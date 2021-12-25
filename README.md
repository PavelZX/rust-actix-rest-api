# rust-actix-rest-api-boilerplate
A Rust RESTful API server with actix-web

## Install
- Install [Rust](https://www.rust-lang.org/)
- Install [Docker](https://www.docker.com/) (optional)
- Install [Docker Compose](https://github.com/docker/compose/releases) (optional)


Install first [cargo-make](https://github.com/sagiegurari/cargo-make)，Can use cargo make command to start some scripts

```
$ cargo install --no-default-features --force cargo-make
```

```
$ git clone address of this library
```

## Start the development environment
```
$ cargo make dev
```

Port 8080 is enabled by default，accessible http://localhost:8080/hello test whether the startup is successful


## Clean up
```
$ cargo make clean
```

## Bale
```
$ cargo make build
```

## mac cross compile linux
```
$ cargo make buildlinux
```

## Start the local development environment database (Docker, optional)
````
$ docker-compose up
````

Pass through docker-compose start up postgres and redis，postgres the port is 5432, redis the port is 6379. The database can be operated and debugged through the local client tool connection

## Stop the local development environment database (Docker, optional)
````
$ docker-compose down
````

## Instruction

Conciseness

### About web frameworks actix-web

[actix-web](https://actix.rs/)It is a fast asynchronous web framework under rust。Used by the underlying asynchronous library [Tokio](https://tokio.rs/)，Pay attention to the use of asynchronous development when developing。

Database connection pool，redis examples of connection pools and configuration files，exist actix web pass at startup `app_data` Incoming actix。

```
App::new()
    .app_data(web::Data::new(AppState {
        config: settings.clone(),
        log: logger.clone(),
        db: db_pool.clone(),
        redis: redis_pool.clone(),
    }))
...
```

### Configuration

Configuration library selection [config](https://github.com/mehcode/config-rs)，The configuration file is `data/config/app.toml`，Configuration data can be passed `web::Data` obtain：

```
pub async fn hello(state: web::Data<AppState>) -> Result<web::HttpResponse, error::Error> {
    let name = state.config.get::<String>("app.name").unwrap();
    ...
}
```

### Log 

Log library selection [slog](https://github.com/slog-rs/slog)，supports asynchronous, configured with log file and screen dual output. The log file is `data/logs/app.log`，actix Журнал можно вести следующими способами：

```
pub async fn hello(state: web::Data<AppState>) -> Result<web::HttpResponse, error::Error> {
    ...
    info!(state.log, "hello {}", name);
    ...
}
```

### Database

Database operation library selection [sqlx](https://github.com/launchbadge/sqlx)，In this example, postgres is configured to support various other commonly used databases. accessible `web::Data` Get the database connection pool. Pay attention to the use of asynchronous development/

### Redis

Redis operational library selection [redis](https://github.com/mitsuhiko/redis-rs)，support asynchronous mode, use [mobc](https://github.com/importcjj/mobc) The configured connection pool. accessible `web::Data` Get the redis connection pool. Pay attention to the use of asynchronous development。

```
use redis::AsyncCommands;
...

pub async fn hello(state: web::Data<AppState>) -> Result<web::HttpResponse, error::Error> {
    let mut con = state.redis.get().await.unwrap();
    let val = con.get("my_key").await.unwrap();
    ...
}

```

### Wrong format

use JSON data structure。

```
{"errcode":100203,"errmsg":"Captcha not found"}
```
(We also return similar error codes uniformly for the back-end interfaces of other languages, so that the front-end doesn’t need to care what language the back-end uses, just use a unified error format.)

actix available in `src/lib/error` the error type defined in the output error：

```
use crate::lib::error;
...

return Err(error::new(400001, "name cannot be empty", 422));
...

```

注：Temporarily use the 422 error code to output the error. In the current Actix version, the 400 error will be covered by the default custom error output, which will be resolved later。

### Input verification

For the time being, the third-party library is not used for input verification, and it is processed in a simple way (I follow my usual method of scaffolding in other languages). Some input formats can be judged by the regularity in `src/lib/validator.rs`, and other needs can be added here Verification. In actix, use? To verify and pass errors。

```
validator::uuid(uuid_var, "uuid")?;
validator::not_none(absent_number, "numerical value")?;
...
```

If you need to judge the input string as non-None and automatically convert Option to String，can be used `validator::required_str` function：

```
let name = validator::required_str(name_param, "name")?;
```
