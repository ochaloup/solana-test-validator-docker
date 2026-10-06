
## build docker images

```sh
# solana base image with anchor, rust and nodejs (corepack provides yarn/pnpm); light and photon are not included
docker build . -t <account>/solana-test-validator:solana_3.1.12-anchor_1.2.1
```

Every download is verified against a sha256 pinned in the `Dockerfile`. A version bump means updating the version and its hash together.
The `solana-docker` volume shadows `/opt`: after switching to this image, remove it (`docker volume rm solana-docker`) or the old toolchain stays.

## run solana-test-validator

```sh
docker run -ti --name solana-test-validator -v ~/dev:/working_dir:rw --rm <account>/solana-test-validator:solana_3.1.12-anchor_1.2.1 bash
```

## run anchor test:

```sh
docker exec -w /working_dir/subpath-to-anchor-project -ti solana-test-validator sh -c "anchor test --skip-local-validator"
```
