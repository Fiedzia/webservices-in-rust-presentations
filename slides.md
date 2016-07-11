!SLIDE

## How to create webservices in Rust?

!SLIDE

## About me

* Search engineer at CV-Library (cv-library.co.uk)
* When writing web applications/services, I work mostly with python/django
* Recently I’ve been porting some performance-critical bits from python to rust and deploying them as webservices

Maciej Dziardziel (fiedzia@gmail.com)


!SLIDE

## Iron

* Extensible, Concurrency Focused Web Development in Rust.
* 100% safe code
* Most complete and popular framework for rust (3k stars, 200+ forks)
* for python developers: its closer to bottle or flask then django
* You only pay for what you use



!SLIDE

## Hello world

~~~~{rust}
    extern crate iron;
    
    use iron::prelude::*;
    use iron::status;
    
    fn main() {
        Iron::new(|_: &mut Request| {
    
            Ok(Response::with((status::Ok, "Hello world!")))
    
        }).http("localhost:3000").unwrap();
    }
~~~~

!SLIDE

## Iron provides

* Middlewares
* Extensions 
* Plugins

!SLIDE

## Middleware

* Execute code before/after your handler
* Middleware can modify request/response,     construct its own or return an error

~~~~{rust}
    let mut chain = Chain::new(router);
    chain.link_before(ResponseTimeMiddleware);
    chain.link_after(StatsdMiddleware);

~~~~

!SLIDE

## Bundled middlewares

* Routing
* Mounting (like routing, but removes part of path for handler)
* Serving static files from given directory
* Logging

Its trivial to write your own

!SLIDE

## Routing

~~~~{rust}
    let mut router = Router::new();
    router.get("/", handler);
    router.post("/", handler);
    router.get("/:query", handler);
    router.any("/page", handler);
~~~~



!SLIDE

## Mounting

* Routing binds handler to url pattern.
  Handler will see **whole url**

* Mounting binds handler to url prefix.
  Handler will see **url without prefix**

!SLIDE

## Mounting

~~~~{rust}
    let mut mount = Mount::new();
    mount.mount("/assets", Static::new(Path::new("/srv/assets/")));
~~~~

!SLIDE

## Your own middleware

~~~~{rust}
    pub struct YourMiddleware;
    
    impl BeforeMiddleware for YourMiddleware {
        fn before(&self, request: &mut Request) -> IronResult<()> {
        //…your logic here
        Ok(())
    }
    impl AfterMiddleware for YourMiddleware {
    
    fn after(&self, req: &mut Request, res: Response) ->     IronResult<Response> {
        //… your logic here
    
        Ok(res)
    }
~~~~

!SLIDE

## Extensions

* Extensions allow to attach information to request/response

Example: lets try to log response time

* ...we need to put start time before handler is run
* … and get it out after

!SLIDE

## Extensions - using typemap


~~~~{rust}
    pub struct ResponseTime;
    
    impl typemap::Key for ResponseTime {
        type Value = u64;
    }
    //set
            request.extensions.insert::<ResponseTime>(precise_time_ns());
    //get
            *request.extensions.get::<ResponseTime>().unwrap()
~~~~


!SLIDE

## Plugins

Plugins allow for lazily-evaluated, automatically cached extensions to Requests and Responses

Iron provides plugins for:

* Cookies
* Sessions
* Parsing request params (get/post/form data/json)

!SLIDE

## Using plugins

~~~~{rust}
    let get = match request.get::<UrlEncodedQuery>() {
        Ok(hashmap) => hashmap,
        Err(UrlDecodingError::EmptyQuery) => HashMap::new(),
        Err(_) => {
            return Ok(Response::with((iron::status::BadRequest, "GET params decoding failed")))
        }
    };
~~~~

!SLIDE

## Receiving JSON

~~~~{rust}
    let json_body = request.get::<bodyparser::Json>();
    let json_data = match json_body {
        Ok(Some(json_data)) => json_data,
        Ok(None) => return Ok(Response::with((iron::status::BadRequest, "Missing json data"))),
        Err(err) => {
            return Ok(Response::with((iron::status::BadRequest,
                                      format!("Json parsing failed: {:?}", err))))
        }
    };
~~~~

!SLIDE

## Sending JSON

~~~~{rust}
    let mut json_result: BTreeMap<String, json::Json> = BtreeMap::new();
    json_result.insert("status".to_string(), json::Json::String("ok".to_string()));
    Ok(Response::with((iron::status::Ok,     json::encode(&json::Json::Object(json_result)).unwrap())))
~~~~

!SLIDE

## Under the hood

* Iron
* Hyper
* Rotor
* MIO
* OS

!SLIDE


## Under the hood


* Iron
* Hyper (HTTP client/server)
* Rotor (nicer API for MIO)
* MIO (evented IO using epoll/kqueue)
* OS (epoll/kqueue)

!SLIDE

## Threads

* Up to Iron, all IO is event-based (using single thread)
* When Iron is initialized, we can define limit on how many threads will be used (defaults to amount of your cores)
* Each request will be handled in own thread
* Iron will manage threadpool


!SLIDE

## Whar Iron **doesn't** provide

* Template language
* ORM
* Database handling (just putting database pool into request.extensions worked for me well)
* Any support for assets (beyond serving them)
* Any integration with js


!SLIDE

## Deployment notes

Cargo.toml:

~~~~{ini}
    [profile.release]
    debug = true
~~~~

I am using docker. Inside container there is also uwsgi:

~~~~{ini}
    [uwsgi]
    attach-daemon = /your_rust_app
    env = RUST_BACKTRACE=1
    logger = file:/var/log/uwsgi_error.log
~~~~

!SLIDE

## Links

* [More about web for rust](http://www.arewewebyet.org/)
* [Iron](http://ironframework.io/)
* [Github page](https://github.com/iron/iron)



!SLIDE

## Any questions?
