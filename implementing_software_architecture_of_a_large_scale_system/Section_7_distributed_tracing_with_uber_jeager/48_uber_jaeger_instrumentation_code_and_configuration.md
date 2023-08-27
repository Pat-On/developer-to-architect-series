# Uber Jaeger Instrumentation Code and Configuration


Python Web config
```python
# Jaeger Tracing settings

config = Config(
    config={ # usually read from some yaml config
        'sampler': {
            #  we can set uop here percent of the request for examples (in the production)
            'type': 'const',
            'param': 1,
        },
        # (where the agent is - usually local host - in docker set up it is going to be local)
        'local_agent': {
            'reporting_host': JAEGER_AGENT_HOST,
            'reporting_port': JAEGER_AGENT_PORT,
        },
        'logging': True,
    },
    service_name='WebApp',
    validate=True,
)

TRACER = config.initialize_tracer()
```

## instrumentalization - example

```python
def products(request):
    with tracer.start_span('getProducts') as span:   <------- here
        access_token = get_access_token(request)
        try:
            req_headers = {}
            tracer.inject(span.context, Format.TEXT_MAP, req_headers)
            req_headers['Authorization']='Bearer '+access_token
            req = session.get(settings.PRODUCT_SVC_ORIGIN + PRODUCTS_URI,
                              headers=req_headers, timeout=settings.HTTP_TIMEOUT)
            req.raise_for_status()
        except HTTPError as http_error:
            logger.error("HTTPError communicating with the backend server for get products: %s", http_error)
            return render(request, 'app/error.html',
                          {'message': 'HTTP Error '+str(req.status_code)+' communicating with the backend server.'})
        except Exception as exception:
            logger.error("Error communicating with a backend server for get products: %s", exception)
            logger.error(traceback.format_exc())
            return render(request, 'app/error.html',
                          {'message': 'Error communicating with the backend server.'})

        if req.status_code == 200:
            logger.info('Products fetched | context=%s', req.content.decode())
            products = req.json()
            for product in products:
                product['imageUrl'] = get_product_image_url(product['id'])
            return render(request, 'app/products.html', {'products': products})
        logger.error('Unable to fetch products from service | userId=%s', get_user(request))
        return HttpResponse("Unable to get products")

```

## Last part - docker image - installing tools



## he added dependency in the common library
```xml
    <dependency>
      <groupId>io.opentracing.contrib</groupId>
      <artifactId>opentracing-spring-jaeger-cloud-starter</artifactId>
      <version>${jaegar.version}</version>
    </dependency>
```