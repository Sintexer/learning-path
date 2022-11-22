# Default fetching policy

| Association Type | Default fetching policy |
|------------------|-------------------------|
| @OneToMany       | LAZY                    |
| @ManyToMany      | LAZY                    |
| @ManyToOne       | EAGER                   |
| @OneToMany       | EAGER                   |

# JPA/Hibernate cache

## 1st layer cache

1st layer cache is a session cache which is associated with current transaction. It is enabled by default and used to prevent quering for the same object. It is implemented as key-value Map inside EntityManager.

## 2nd layer cache

2nd layer cache is a global cache that is stored inside SessionFactory and same for every transaction. It could be usefull for read-only object (`@org.hibernate.annotations.Cache(usage = CacheConcurrencyStrategy.READ_ONLY)`). 2nd layer cache disabled by default and could be enabled. You need to add implementation dependency to make it work (e.g., `hibernate-ehcache`).

``` yaml
spring:
  ...
  jpa:
    ...
    properties:
      hibernate:
        cache:
          use_second_level_cache: true
          region:
            factory_class: org.hibernate.cache.ehcache.EhCacheRegionFactory
```

# N+1 problem

https://habr.com/ru/company/otus/blog/529692/

There are several possible N+1 issues when using JPA and Hibernate.

## FetchType.EAGER

EAGER is bad for memory and performance in most cases.

@ManyToOne and @OneToOne use FetchType.EAGER by default. In case of such mapping:

``` java
@ManyToOne
private Post post;
```

You use FetchType.EAGER. And every time you forget JOIN FETCH when loading PostComment using JPQL or Criteria API:

``` java
List<PostComment> comments = entityManager
.createQuery("""
    select pc
    from PostComment pc
    """, PostComment.class)
.getResultList();
```

You will face N+1 :

``` SQL
SELECT
    pc.id AS id1_1_,
    pc.post_id AS post_id3_1_,
    pc.review AS review2_1_
FROM
    post_comment pc
 
SELECT p.id AS id1_0_0_, p.title AS title2_0_0_ FROM post p WHERE p.id = 1
SELECT p.id AS id1_0_0_, p.title AS title2_0_0_ FROM post p WHERE p.id = 2
SELECT p.id AS id1_0_0_, p.title AS title2_0_0_ FROM post p WHERE p.id = 3
SELECT p.id AS id1_0_0_, p.title AS title2_0_0_ FROM post p WHERE p.id = 4
```

One select for each PostComment to fetch related Post.

Use JOIN FETCH to avoid N+1:

``` java
List<PostComment> comments = entityManager.createQuery("""
    select pc
    from PostComment pc
    join fetch pc.post p
    """, PostComment.class)
.getResultList();
````
 
In this case you'll have only one SQL query:

``` SQL
SELECT
    pc.id as id1_1_0_,
    pc.post_id as post_id3_1_0_,
    pc.review as review2_1_0_,
    p.id as id1_0_1_,
    p.title as title2_0_1_
FROM
    post_comment pc
INNER JOIN
    post p ON pc.post_id = p.id
```

[Read why JPA and Hibernate FetchType.EAGER is a code smell](https://vladmihalcea.com/eager-fetching-is-a-code-smell/)

## FetchType.LAZY

N+1 problem with `FetchType.LAZY` is pretty obvious - you fetch collection of comments and they are loaded without post, but if you'll traverse through comment and get their posts, for each post-postComment you'll have a select post query.

## N+1 for biderectional @OneToOne

Imagine you have Post and PostDetails, that share same primary key. In this key PostDetails primery key is a foreign key for Post.

When using unidirectional @OneToOne there is no problem. Using @mapsId annotation on PostDetails side you easily can have such relation. 

But when adding biderectional association you could face the problem when using `FetchType.LAZY` on the Parent side if you use advanced setter, where a biderectional association is initialized:

``` java

public void setDetails(PostDetails details) {
    this.details = details;
    details.post = this;
}
```

The line `details.post = this;` will lead to second SELECT when quering for post - hibernate will try to inject a proxy of PostDetails and this eventually will trigger proxy to be loaded.

This can be avoided with Bytecode enhancement like `@LazyToOneOption.NO_PROXY`, which force you to remove `@MapsId` on the child side as well.

But also when using `@MapsId` there is no much sense for biderectional assocoation since Post and PostDetails share the same `id`, so you can easily fetch PostDetails having only Post.

By default the name of the column of the field with `@MapsId` annotation will be field name + `_id`, but if you want to change it (e.g., to `id`, at least because it is a primary key) use good old `@JoinColumn(name = "desired_column_name")`
