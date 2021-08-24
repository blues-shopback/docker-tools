ARG image
FROM ${image}

COPY . /docker_setting
WORKDIR /docker_setting

# Jupyter config
RUN pip install --upgrade pip && \
    pip install jupyterlab && \
    mkdir -p /root/.jupyter/ && \
    cp -r /docker_setting/jupyter/* /home/work/.jupyter/ && \
    cp -r /docker_setting/jupyter/* /root/.jupyter/ && \
    cp .bashrc /root/

ENTRYPOINT bash
