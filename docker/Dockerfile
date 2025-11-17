FROM ubuntu:22.04

# Global environment
ENV TERM=xterm-256color
ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update

# Basics
RUN apt-get install -y --no-install-recommends ca-certificates  git wget unzip
RUN apt-get install -y --no-install-recommends curl
RUN apt-get install -y --no-install-recommends git
RUN apt-get install -y --no-install-recommends wget
RUN apt-get install -y --no-install-recommends unzip

# Shell
RUN apt-get install -y --no-install-recommends zsh
RUN chsh -s $(which zsh)

# Build tools
RUN apt-get install -y --no-install-recommends build-essential
RUN apt-get install -y --no-install-recommends cmake
RUN apt-get install -y --no-install-recommends software-properties-common

# SDK dependencies
RUN apt-get install -y --no-install-recommends libopencv-dev
RUN apt-get install -y --no-install-recommends libboost-all-dev
RUN apt-get install -y --no-install-recommends libusb-1.0-0-dev
RUN apt-get install -y --no-install-recommends libprotobuf-dev
RUN apt-get install -y --no-install-recommends protobuf-compiler
RUN apt-get install -y --no-install-recommends libhdf5-dev
RUN apt-get install -y --no-install-recommends hdf5-tools
RUN apt-get install -y --no-install-recommends libglew-dev
RUN apt-get install -y --no-install-recommends libglfw3-dev
RUN apt-get install -y --no-install-recommends libcanberra-gtk-module
RUN apt-get install -y --no-install-recommends ffmpeg
RUN apt-get install -y --no-install-recommends tmux
RUN apt-get install -y --no-install-recommends mesa-utils
RUN apt-get install -y --no-install-recommends usbutils
RUN apt-get install -y --no-install-recommends udev
RUN apt-get install -y --no-install-recommends python3.10
RUN apt-get install -y --no-install-recommends python3-pip
RUN apt-get install -y --no-install-recommends python3.10-venv

# Setup shell: oh-my-zsh, p10k, plugins
RUN sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" -- -y
RUN git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
RUN git clone --depth=1 https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-/root/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
RUN git clone --depth=1 https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-/root/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
RUN sed -i 's/ZSH_THEME="robbyrussell"/ZSH_THEME="powerlevel10k\/powerlevel10k"/' /root/.zshrc

COPY docker/config/.p10k.zsh docker/config/.zshrc /root/

# Setup python environment
RUN python3 -m venv /opt/prophesee/psee-py3venv --system-site-packages
RUN ln -s /usr/bin/python3.10 /usr/bin/python

ENV PATH="/opt/prophesee/psee-py3venv/bin:$PATH"

# Install pip dependencies
RUN pip install --upgrade pip
COPY requirements.txt /tmp/
RUN pip install -r /tmp/requirements.txt
RUN rm -rf /root/.cache/pip /tmp/requirements.txt

# Install pybind11 v2.11.0 (system-wide)
WORKDIR /opt
RUN wget -q https://github.com/pybind/pybind11/archive/v2.11.0.zip
RUN unzip v2.11.0.zip
RUN cd pybind11-2.11.0
RUN mkdir build
WORKDIR /opt/pybind11-2.11.0/build
RUN cmake .. -DPYBIND11_TEST=OFF
RUN cmake --build . --config Release
RUN cmake --install .
RUN cd /opt
RUN rm -rf pybind11-2.11.0 v2.11.0.zip

# Build Prophesee OpenEB SDK (branch 5.1.1)
RUN export PYTHONNOUSERSITE=true
RUN git clone --branch 5.1.1 --depth 1 https://github.com/prophesee-ai/openeb.git /opt/openeb
RUN pip install -r /opt/openeb/utils/python/requirements_openeb.txt
RUN cd /opt/openeb
RUN mkdir build
WORKDIR /opt/openeb/build
RUN cmake .. \
    -DCMAKE_BUILD_TYPE=Release \
    -DBUILD_TESTING=OFF \
    -DCOMPILE_PYTHON3_BINDINGS=ON \
    -DPython3_EXECUTABLE=$(which python3)
RUN cmake --build . --config Release -- -j$(nproc)
RUN cmake --build . --target install

ENV LD_LIBRARY_PATH=""
ENV LD_LIBRARY_PATH="/opt/openeb/build/lib:${LD_LIBRARY_PATH}"

# Install the project
COPY setup.py /root/setup.py
COPY src /root/src
WORKDIR /root
RUN python -m pip install -e .
RUN rm setup.py

# Entrypoint
COPY bash/entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]

# Final cleanup
RUN apt-get purge -y --auto-remove cmake
RUN apt-get purge -y --auto-remove unzip
RUN apt-get purge -y --auto-remove ca-certificates
RUN rm -rf /var/lib/apt/lists/* /root/.cache /opt/*.zip

WORKDIR /root
