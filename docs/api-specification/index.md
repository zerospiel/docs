---
hide:
  - navigation
  - toc
---

<!doctype html>
<html>
  <head>
    <title>k0rdent v1.5.0 API documentation</title>                                                            <!-- page title here -->
    <meta charset="utf-8"/>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <style>
      body { margin: 0; padding: 0; }
    </style>
  </head>
  <body>
    <redoc
           lazy-rendering
           >
    </redoc>
    <script src="https://cdn.redoc.ly/redoc/latest/bundles/redoc.standalone.js"></script>
    <!-- <script src="../javascripts/redoc.js"></script> -->                                                <!-- IGNORE THIS -->
    
    <script>
        
        var spec = "../openapi/k0rdent-api-1.5.0.json";                                                          <!-- path to spec here -->                                          
        
        Redoc.init(spec);
    </script>
  </body>
</html>