# AWS Lambda [Rust](https://www.rust-lang.org/) docker builder 🐑 🐳 🦀

## 🤔 about

This docker image extends [lambda ci `provided`](https://github.com/lambci/docker-lambda#documentation) builder docker image, a faithful reproduction of the actual AWS "**provided**" Lambda runtime environment,
and installs [rustup](https://rustup.rs/) and the *stable* rust toolchain.

## 📦 install

Tags for this docker image follow the naming convention `softprops/lambda-rust:{version}-rust-{rust-stable-version}`
where `{rust-stable-version}` is a stable version of rust.

You can find a list of available docker tags [here](https://hub.docker.com/r/softprops/lambda-rust/)

> 💡 If you don't find the version you're looking for, please open a new github issue to publish one

You can also depend directly on `softprops/lambda-rust:latest` for the most recently published version.

## 🤸 usage

The default docker entrypoint will build a packaged release optimized version your Rust artifact under `target/lambda/release` to
isolate the lambda specific build artifacts from your host-local build artifacts.

You will want to volume mount `/code` to the directory containing your cargo project.

You can pass additional flags to `cargo`, the Rust build tool, by setting the `CARGO_FLAGS` docker env variable

A typical docker run might look like the following.

```bash
$ docker run --rm \
	-v ${PWD}:/code \
	-v ${HOME}/.cargo/registry:/root/.cargo/registry \
	-v ${HOME}/.cargo/git:/root/.cargo/git \
	softprops/lambda-rust
```

> 💡 The -v (volume mount) flags for `/root/.cargo/{registry,git}` are optional but when supplied, provides a much faster turn around when doing iterative development

## 🔬 local testing

Once you've built a Rust lambda function artifact, the `provided` runtime expects
deployments of that artifact to be named "**bootstrap**". The `lambda-rust` docker image
builds a zip file, named after the binary, containing your binary files renamed to "bootstrap"

You can invoke this bootstap executable with the lambda-ci docker image for the `provided` AWS lambda runtime.

```sh
# start a docker container replicating the "provided" lambda runtime
# awaiting an event to be provided via stdin
$ unzip \
    target/lambda/release/{your-binary-name}.zip \
    -d /tmp/lambda && \
  docker run \
    -i -e DOCKER_LAMBDA_USE_STDIN=1 \
    --rm \
    -v /tmp/lambda:/var/task \
    lambci/lambda:provided

# provide an event payload via stdin (typically a json blob)

# Ctrl-D to yield control back to your function
```