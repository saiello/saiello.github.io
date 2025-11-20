+++
title = 'Gomemlimit base image'
date = 2025-11-20T09:34:00+01:00
draft = false
tags = ['golang', 'docker', 'til']
+++

Hey there, developers!

I want to share a simple proof of concept on [GitHub](https://github.com/saiello/gomemlimit-base-image) demonstrating how automatically configure the Go runtime's GOMEMLIMIT based on the memory available to the container.

If you’re not familiar with GOMEMLIMIT, you can read this [blog post](https://weaviate.io/blog/gomemlimit-a-game-changer-for-high-memory-applications).


## How it works

When the container starts:

- It detects the maximum memory available to the container.
- It applies a ratio (default: 0.8) to determine a safe memory limit for the Go runtime.
- It exports GOMEMLIMIT accordingly before your application launches.

This makes the image plug-and-play for any Go application that benefits from smarter memory limiting.

## Usage


Clone and build the image

```
git clone https://github.com/saiello/gomemlimit-base-image
cd gomemlimit-base-image
make
```

```
docker run -e GOMEMLIMIT_RATIO=0.7 --memory 20000000 gomemlimit-base-image
```

## Alternative

There is an alternative approach: using a [Go library](https://github.com/KimMachineGun/automemlimit) that determines available memory and sets GOMEMLIMIT programmatically inside your application.

## Resouces

- [saiello/gomemlimit-base-image](https://github.com/saiello/gomemlimit-base-image)
- [KimMachineGun/automemlimit](https://github.com/KimMachineGun/automemlimit)
- [Gomemlimit Blog Post](https://weaviate.io/blog/gomemlimit-a-game-changer-for-high-memory-applications)