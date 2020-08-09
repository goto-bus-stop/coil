# Coil
An async task queue built with SQLx, Postgres, and Rayon


##### This software is alpha, and not intended for production use yet

Coil is built first for use in [`substrate-archive`](https://github.com/paritytech/substrate-archive) and takes heavily from [swirl](https://github.com/sgrif/swirl)



Supports synchronous and asynchronous jobs. Synchronous jobs will be spawned into a threadpool managed by [`rayon`](https://github.com/rayon-rs/rayon). Async jobs will be spawned onto an executor. The only requirement is that the executor implements the futures `Spawn` trait. This way, `coil` supports `Tokio`, `smol`, and `async-std`.



### Examples

```rust
struct Size {
	width: u32,
	height: u32
}

#[coil::background_task]
async fn resize_image(id: u32, size: Size) -> Result<(), Error> {
	// some work
}
```

With an environment
```rust
struct Size {
	width: u32,
	height: u32
}

struct Environment {
    file_server_private_key: String,
    http_client: http_lib::Client,
    conn: sqlx::PgPool
}

#[coil::background_task]
async fn resize_image(env: &Environment, id: u32, size: Size) -> Result<(), Error> {
	// some work
}
```

```rust
	resize_image_with_env("tohru".to_string(), Size { height: 32, width: 32 }).enqueue(&pool).await;
	let runner = coil::RunnerBuilder::new(env, Executor, pool)
            .num_threads(8)
            .build()
            .unwrap();
	runner.run_all_pending_tasks().await.unwrap()
```

### Differences from [`swirl`](https://github.com/sgrif/swirl)
- Supports asyncronous jobs/executors
- Serializes data into Postgres with Messagepack instead of Json. Saves disk-space but doesn't allow for querying of job-data directly in SQL
- In asyncronous jobs, database queries will be run asynchronously with SQLx
- Migrations are included in the binary and exposed via a `migrate` fn. Results in less boilerplate setting up the database.
- Enqueue is an async fn
