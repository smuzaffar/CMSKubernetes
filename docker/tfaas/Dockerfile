FROM cern/cc7-base
MAINTAINER Valentin Kuznetsov vkuznet@gmail.com

ENV WDIR=/data
EXPOSE 8083
ADD tfaas_start.sh $WDIR/tfaas_start.sh
ADD config.json $WDIR/config.json

RUN yum update -y && yum clean all
RUN yum install -y git-core krb5-devel readline-devel openssl autoconf automake libtool make gcc gcc-c++ unzip
RUN yum clean all

# start the setup
RUN mkdir -p $WDIR
WORKDIR ${WDIR}

# download golang and install it
RUN curl -k -L -O https://dl.google.com/go/go1.9.4.linux-amd64.tar.gz
RUN tar xfz go1.9.4.linux-amd64.tar.gz
RUN mkdir $WDIR/gopath
RUN rm go1.9.4.linux-amd64.tar.gz
ENV GOROOT=$WDIR/go
ENV GOPATH=$WDIR/gopath
ENV PATH="${GOROOT}/bin:${WDIR}:${PATH}"

# install Go dependencies
RUN go get github.com/dmwm/cmsauth
RUN go get github.com/vkuznet/x509proxy
RUN go get github.com/sirupsen/logrus

# download and insta TensorFlow libraries
# https://www.tensorflow.org/versions/master/install/install_go
ENV TF_LIB="libtensorflow-cpu-linux-x86_64-1.6.0.tar.gz"
RUN curl -k -L -O "https://storage.googleapis.com/tensorflow/libtensorflow/${TF_LIB}"
RUN tar xfz $TF_LIB
ENV LIBRARY_PATH="${LIBRARY_PATH}:${WDIR}/lib"
ENV LD_LIBRARY_PATH="${LD_LIBRARY_PATH}:${WDIR}/lib"
RUN go get github.com/tensorflow/tensorflow/tensorflow/go
RUN go get github.com/tensorflow/tensorflow/tensorflow/go/op

# install protobuf
WORKDIR ${WDIR}
RUN git clone https://github.com/google/protobuf.git
WORKDIR ${WDIR}/protobuf
RUN ./autogen.sh
RUN ./configure --prefix=${WDIR}
RUN make
RUN make install
RUN go get -u github.com/golang/protobuf/protoc-gen-go
#RUN go get github.com/golang/protobuf

# build tfaas
WORKDIR ${WDIR}
RUN git clone https://github.com/vkuznet/TFaaS.git
WORKDIR $WDIR/TFaaS/src/Go
RUN make
ENV X509_USER_PROXY=/etc/secrets/tfaas-proxy
ENV PATH="${WDIR}/TFaaS/src/Go:${PATH}"

# run the service
WORKDIR ${WDIR}
RUN mkdir models
RUN touch labels.csv
RUN rm ${TF_LIB}
RUN cat $WDIR/config.json | sed -e "s,GOPATH,$GOPATH,g" > $WDIR/tfaas-config.json
CMD ["tfaas_start.sh"]