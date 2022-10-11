ARG IMAGE_NAME
FROM ${IMAGE_NAME}:dev AS builder

ENV SRC_DIR=/usr/src            \
    DIST_DIR=/dist              \
    LINUX_DIR=/usr/src/linux    \
    LINUX_REPO_URL=git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git

ARG KERNEL_VERSION

RUN mkdir -p ${SRC_DIR} ${DIST_DIR} && \
    git clone --depth 1 --branch v${KERNEL_VERSION} ${LINUX_REPO_URL} ${LINUX_DIR} && \
    cd ${LINUX_DIR}

WORKDIR ${LINUX_DIR}

COPY configs/linux-x86_64-${KERNEL_VERSION}.config .config

RUN apt install -y python3 cpio
RUN make olddefconfig
RUN make modules_prepare
# -j32 tells the compiler to use 32 threads, if 32 threads are not available it will use as many as it can up to 32
RUN make -j32
RUN make modules_install
RUN make headers_install

RUN cp vmlinux /boot/vmlinux && \
    cp .config /boot/config-${KERNEL_VERSION}

# FROM ubuntu:20.04
FROM scratch
COPY --from=builder /boot /boot
COPY --from=builder /lib/modules /lib/modules
COPY --from=builder /usr/include /usr/include
COPY --from=builder /usr/src/linux /usr/src/linux
