# Default fetching policy

| Association Type | Default fetching policy |
|------------------|-------------------------|
| @OneToMany       | LAZY                    |
| @ManyToMany      | LAZY                    |
| @ManyToOne       | EAGER                   |
| @OneToMany       | EAGER                   |

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
