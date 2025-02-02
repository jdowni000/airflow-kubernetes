ARG BASE_IMAGE=quay.io/cloud-bulldozer/airflow:2.2.3
FROM ${BASE_IMAGE} as runtime
USER root
RUN curl -L https://go.dev/dl/go1.17.6.linux-amd64.tar.gz -o go1.17.6.linux-amd64.tar.gz
RUN tar -C /usr/local -xzf go1.17.6.linux-amd64.tar.gz
ENV PATH=$PATH:/usr/local/go/bin
RUN mkdir -p /home/airflow/go
ENV GOPATH=/home/airflow/go
RUN go install github.com/markbates/pkger/cmd/pkger@latest
RUN mv $GOPATH/bin/pkger /usr/local/bin/
ENV GOFLAGS=
RUN apt update -y
RUN env ACCEPT_EULA=Y apt upgrade -y
RUN apt install jq unzip bsdmainutils build-essential libncurses5-dev zlib1g-dev libnss3-dev libgdbm-dev libssl-dev libsqlite3-dev libffi-dev libreadline-dev curl libbz2-dev -y
RUN curl -L $(curl -s https://api.github.com/repos/openshift/rosa/releases/latest | jq -r ".assets[] | select(.name == \"rosa-linux-amd64\") | .browser_download_url") --output /usr/local/bin/rosa
RUN curl -L $(curl -s https://api.github.com/repos/openshift-online/ocm-cli/releases/latest | jq -r ".assets[] | select(.name == \"ocm-linux-amd64\") | .browser_download_url") --output /usr/local/bin/ocm
RUN chmod +x /usr/local/bin/rosa && chmod +x /usr/local/bin/ocm
RUN /usr/local/bin/rosa download openshift-client
RUN tar xzvf openshift-client-linux.tar.gz
RUN curl "https://www.python.org/ftp/python/3.9.9/Python-3.9.9.tar.xz" | tar xJf -
RUN cd Python-3.9.9 && ./configure --enable-optimizations && make && make altinstall
RUN echo "export CLOUDSDK_PYTHON=python3.9" >> /home/airflow/.bashrc
RUN echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] http://packages.cloud.google.com/apt cloud-sdk main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list && curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key --keyring /usr/share/keyrings/cloud.google.gpg  add - && apt-get update -y && apt-get install google-cloud-sdk -y
RUN mv oc kubectl /usr/local/bin/
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
RUN unzip awscliv2.zip
RUN ./aws/install
RUN curl -sL https://aka.ms/InstallAzureCLIDeb | bash
RUN echo "airflow ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
RUN git clone --branch main https://github.com/openshift/hypershift
WORKDIR hypershift
RUN make build
WORKDIR /opt/airflow/
USER airflow
RUN python3 -m pip install --upgrade pip || true
RUN yes | pip3 install openshift==0.11.0 || true
