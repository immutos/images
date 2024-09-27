VERSION 0.8
FROM debian:bookworm
WORKDIR /workspace
ENV DO_NOT_TRACK=1

build:
  FROM +tools
  COPY *.yaml ./
  WITH DOCKER --allow-privileged
    RUN --privileged for recipe in $(find . -name 'bookworm*.yaml'); do \
        debco build -p linux/amd64,linux/arm64 -o "$(basename "$recipe" .yaml).tar" -f "$recipe"; \
      done \
      && for recipe in $(find . -name 'trixie*.yaml'); do \
        debco build -p linux/amd64,linux/arm64,linux/riscv64 -o "$(basename "$recipe" .yaml).tar" -f "$recipe"; \
      done
  END
  RUN for image in $(find . -name '*.tar'); do \
      skopeo inspect "oci-archive:${image}" | jq -j '.Digest' >> digests.txt; \
      echo " registry.dpeckett.dev/debian:$(basename "$image" .tar)" >> digests.txt; \
    done
  SAVE ARTIFACT *.tar AS LOCAL dist/
  SAVE ARTIFACT digests.txt AS LOCAL dist/digests.txt

push:
  FROM +tools
  RUN mkdir -p dist
  COPY dist/*.tar ./dist/
  RUN --secret REGISTRY_PASSWORD=registry_password echo "${REGISTRY_PASSWORD}" | docker login -u github-actions --password-stdin registry.dpeckett.dev \
    && for image in $(find . -name '*.tar'); do \
      skopeo copy --multi-arch all "oci-archive:${image}" "docker://registry.dpeckett.dev/debian:$(basename "$image" .tar)"; \
    done

tools:
  RUN apt update \
    && apt install -y curl \
    && curl -fL -o /etc/apt/keyrings/apt-immutos-com-keyring.asc https://apt.immutos.com/signing_key.asc \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/apt-immutos-com-keyring.asc] http://apt.immutos.com bookworm stable" > /etc/apt/sources.list.d/apt-immutos-com.list \
    && apt update
  RUN apt install -y jq qemu-user-binfmt libgpgme11 debco
  RUN curl -fsSL https://get.docker.com | bash
  COPY +skopeo/default-policy.json /etc/containers/policy.json
  COPY +skopeo/skopeo /usr/local/bin/skopeo

# The version in apt is too old, so we need to build from source.
skopeo:
  FROM golang:1.22-bookworm
  WORKDIR /workspace
  RUN apt update
  RUN apt install -y make libgpgme-dev libassuan-dev libbtrfs-dev pkg-config
  ARG VERSION=1.16.1
  RUN curl -fsL "https://github.com/containers/skopeo/archive/refs/tags/v${VERSION}.tar.gz" | tar -xz
  WORKDIR /workspace/skopeo-${VERSION}
  RUN DISABLE_DOCS=1 make
  SAVE ARTIFACT default-policy.json
  SAVE ARTIFACT bin/skopeo
