FROM registry.aliyuncs.com/acs-sample/jenkins-slave-dind:0.1

MAINTAINER fffz <Milelf@gmail.com>

# remove several traces of debian python
RUN apt-get purge -y python.*
# http://bugs.python.org/issue19846
# > At the moment, setting "LANG=C" on a Linux system *fundamentally breaks Python 3*, and that's not OK.
ENV LANG C.UTF-8

ENV PYTHON_VERSION 3.6.6

# if this is called "PIP_VERSION", pip explodes with "ValueError: invalid truth value '<VERSION>'"
ENV PYTHON_PIP_VERSION 10.0.1
#
RUN set -ex \
	&& curl -fSL "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz" -o python.tar.xz \
	&& curl -fSL "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz.asc" -o python.tar.xz.asc \
	&& curl -fSL "https://www.python.org/static/files/pubkeys.txt" -o pubkeys.txt \
	&& gpg --import pubkeys.txt \
	&& gpg --verify python.tar.xz.asc \
	&& mkdir -p /usr/src/python \
	&& tar -xJC /usr/src/python --strip-components=1 -f python.tar.xz \
	&& rm -f python.tar.xz* pubkeys.txt \
	\
	&& cd /usr/src/python \
	&& ./configure --enable-shared --enable-unicode=ucs4 \
	&& make -j$(nproc) \
	&& make install \
	&& ldconfig
RUN set -ex \
	# && curl -fSL 'https://bootstrap.pypa.io/get-pip.py' | python3 \
	# && pip install --no-cache-dir --upgrade pip==$PYTHON_PIP_VERSION \
	&& find /usr/local -depth \
		\( \
		    \( -type d -a -name test -o -name tests \) \
		    -o \
		    \( -type f -a -name '*.pyc' -o -name '*.pyo' \) \
		\) -exec rm -rf '{}' + \
	&& rm -rf /usr/src/python ~/.cache

# install "virtualenv", since the vast majority of users of this image will want it
COPY pip.conf /root/.pip/pip.conf
RUN python3 -m pip install --no-cache-dir virtualenv selenium chromedriver

RUN set -ex \
	&& echo 'deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main' >> /etc/apt/sources.list \
	&& curl -fSL "https://dl.google.com/linux/linux_signing_key.pub" -o linux_signing_key.pub \
	&& apt-key add linux_signing_key.pub \
	&& apt-get update \
	&& apt-get install -y google-chrome-stable \
	&& rm -f /etc/apt/sources.list.d/google-chrome.list \
	&& rm -f linux_signing_key.pub

ENV CHROMEDRIVER_VERSION 2.40
RUN set -ex \
	&& apt-get install -y zip \
	&& apt-get autoremove -y \
	&& curl -fSL "http://chromedriver.storage.googleapis.com/${CHROMEDRIVER_VERSION}/chromedriver_linux64.zip" -o chromedriver.zip \
	&& unzip chromedriver.zip -d /opt/chromedriver \
	&& ln -s /opt/chromedriver/chromedriver /usr/bin/ \
	&& rm -f chromedriver.zip

RUN set -ex \
	&& locale-gen zh_CN.UTF-8 \
	&& update-locale LANG=zh_CN.UTF-8

CMD ["/usr/sbin/sshd", "-D"]
