FROM jetbrains/teamcity-agent

USER root

# Install sudo
RUN apt-get update \
	&& apt-get install -y --no-install-recommends sudo wget \
	&& apt-get clean \
	&& rm -rf /var/lib/apt/lists/* \
	&& echo 'buildagent ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers

# Install .NET 5
RUN cd /tmp \
	&& wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb \
	&& dpkg -i packages-microsoft-prod.deb \
	&& apt-get update \
	&& apt-get install -y apt-transport-https \
	&& apt-get update \
	&& apt-get install -y dotnet-sdk-5.0

# Install clang 11.0.0
RUN apt-get update \
	&& apt-get install -y xz-utils build-essential curl wget \
	&& rm -rf /var/lib/apt/lists/* \
	&& cd /tmp \
	&& curl -SL https://github.com/llvm/llvm-project/releases/download/llvmorg-11.0.0/clang+llvm-11.0.0-x86_64-linux-gnu-ubuntu-20.04.tar.xz | tar -xJC . \
	&& mv clang+llvm-11.0.0-x86_64-linux-gnu-ubuntu-20.04 /opt/clang_11.0.0 \
	&& chown -R root:root /opt/clang_11.0.0

USER buildagent

RUN echo 'export PATH=/opt/clang_11.0.0/bin:$PATH' >> ~/.bashrc \
	&& echo 'export LD_LIBRARY_PATH=/opt/clang_11.0.0/lib:$LD_LIBRARY_PATH' >> ~/.bashrc

