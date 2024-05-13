In conjunction with [[AWS API Gateway data mapping|data transformation]] and [[AWS API Gateway error handling|customized Gateway Responses]], you can also let API Gateway handle some of your basic validations, rather than making the call or building that validation into the backend. 

API Gateway verifies either or both of the following conditions for you.

- The required request parameters in the URL, query string, and headers of an incoming request are included and non-blank. 
- The applicable request payload adheres to the configured JSON request model of the method.