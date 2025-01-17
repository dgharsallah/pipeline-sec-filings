# syntax=docker/dockerfile:experimental

from centos:centos7.9.2009

# NOTE(crag): NB_USER ARG for mybinder.org compat:
#             https://mybinder.readthedocs.io/en/latest/tutorials/dockerfile.html
ARG NB_USER=notebook-user
ARG NB_UID=1000
ARG PIP_VERSION

RUN yum -y update && \
  yum -y install gcc openssl-devel bzip2-devel libffi-devel make git sqlite-devel && \
  curl -O https://www.python.org/ftp/python/3.8.14/Python-3.8.14.tgz && tar -xzf Python-3.8.14.tgz && \
  cd Python-3.8.14/ && ./configure --enable-optimizations && make altinstall && \
  cd .. && rm -rf Python-3.8.14*

# create user with a home directory
ENV USER ${NB_USER}
ENV HOME /home/${NB_USER}

RUN groupadd --gid ${NB_UID} ${NB_USER}
RUN useradd --uid ${NB_UID}  --gid ${NB_UID} ${NB_USER}
USER ${NB_USER}
WORKDIR ${HOME}
RUN mkdir ${HOME}/.ssh && chmod go-rwx ${HOME}/.ssh \
  &&  ssh-keyscan -t rsa github.com >> /home/${NB_USER}/.ssh/known_hosts

ENV PYTHONPATH="${PYTHONPATH}:${HOME}"
ENV PATH="/home/${NB_USER}/.local/bin:${PATH}"

COPY requirements/dev.txt requirements-dev.txt
COPY requirements/base.txt requirements-base.txt
COPY prepline_sec_filings prepline_sec_filings

# NOTE(robinson) - Can remove the secret mount once the unstructured repo is public
# NOTE(crag) - Cannot use an ARG in the dst= path (so it seems), hence no ${NB_USER}, ${NB_UID}
RUN --mount=type=secret,id=ssh_key,uid=1000,dst=/home/notebook-user/.ssh/id_rsa \
  python3.8 -m pip install --no-cache -r requirements-base.txt \
  && python3.8 -m pip install --no-cache -r requirements-dev.txt \
  && python3.8 -c "import nltk; nltk.download('punkt')" \
  && python3.8 -c "import nltk; nltk.download('averaged_perceptron_tagger')"

EXPOSE 5000

ENTRYPOINT ["uvicorn", "prepline_sec_filings.api.section:app", \
  "--host", "0.0.0.0", \
  "--port", "5000"]
