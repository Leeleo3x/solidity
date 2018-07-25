FROM ubuntu:16.04
MAINTAINER Leo Li <leeleo3x@gmail.com>
#Official solidity docker image

#Establish working directory as solidity
WORKDIR /solidity

RUN apt-get -y update && \
	DEBIAN_FRONTEND=noninteractive apt-get -y install python-pip git && \
	git clone https://github.com/Leeleo3x/manticore.git && \
	cd manticore && \
	pip install .
RUN apt-get install -y build-essential software-properties-common && \
  add-apt-repository -y ppa:ethereum/ethereum && \
  apt-get update && \
  apt-get install -y ethereum sudo
# Build dependencies
ADD /scripts/install_deps.sh /solidity/scripts/install_deps.sh
RUN ./scripts/install_deps.sh

#Copy working directory on travis to the image
COPY / $WORKDIR

# Number of parallel jobs during build
# or 0 for auto-computing (max(1, CPU_core_count * 2/3), a greedy value)
ARG BUILD_CONCURRENCY="0"

#Install dependencies, eliminate annoying warnings
RUN sed -i -E -e 's/include <sys\/poll.h>/include <poll.h>/' /usr/include/boost/asio/detail/socket_types.hpp
RUN cmake -DCMAKE_BUILD_TYPE=Release -DTESTS=0 -DSOLC_LINK_STATIC=1
RUN make solc \
    -j$(awk "BEGIN {                                       \
        if (${BUILD_CONCURRENCY} != 0) {                   \
            print(${BUILD_CONCURRENCY});                   \
        } else {                                           \
            x=($(grep -c ^processor /proc/cpuinfo) * 2/3); \
            if (x > 1) {                                   \
                printf(\"%d\n\", x);                       \
            } else {                                       \
                print(1);                                  \
            }                                              \
        }                                                  \
    }")
RUN make install

CMD ["/bin/bash"]
