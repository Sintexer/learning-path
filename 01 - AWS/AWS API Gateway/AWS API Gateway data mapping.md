Mapping templates can be added to the integration request to transform the incoming request to the format required by the backend of the application or to transform the backend payload to the format required by the method response.

For example, you can set up JSON to XML conversion. 

`$input` variable contains body, JSON, params, path. `$stageVariable` - any stage variable name. `$util` can be used to escape JS, parse JSON, encode or decode a URL, etc.