# Stage 1: Build the application
FROM rust:1.81 AS builder

ENV MADARA_VER="fix/block-0-handling"

# Install build dependencies
RUN apt-get -y update && \
  apt-get install -y clang && \
  apt-get autoremove -y; \
  apt-get clean; \
  rm -rf /var/lib/apt/lists/*

# Install Python using apt
SHELL ["/bin/bash", "-c"]

RUN curl -fsSL https://pyenv.run | bash
ENV PYENV_ROOT="/root/.pyenv"
ENV PATH="$PYENV_ROOT/bin:$PYENV_ROOT/shims:$PATH"

RUN eval "$(pyenv init -)" && \
  pyenv install 3.9 && \
  pyenv global 3.9

# Set the working directory
WORKDIR /usr/src/madara/


RUN echo Cloning madara with branch ${MADARA_VER}
RUN git clone --branch ${MADARA_VER} --single-branch https://github.com/madara-alliance/madara . && \
  git submodule update --init --recursive

RUN make snos

RUN apt-get update && \
    apt-get install -y software-properties-common && \
    add-apt-repository -y ppa:ethereum/ethereum && \
    apt-get update && \
    apt-get install -y solc

# Build the application in release mode
RUN cargo build --release
# Stage 2: Create the final runtime image
FROM debian:bookworm
# Install runtime dependencies
RUN apt-get -y update && \
  apt-get install -y openssl ca-certificates tini curl && \
  apt-get autoremove -y; \
  apt-get clean; \
  rm -rf /var/lib/apt/lists/*
# Set the working directory
WORKDIR /usr/local/bin
# Copy the compiled binary from the builder stage
COPY --from=builder /usr/src/madara/target/release/madara .

# Set the entrypoint
ENTRYPOINT ["./madara"]
