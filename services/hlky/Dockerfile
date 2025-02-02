# syntax=docker/dockerfile:1

FROM continuumio/miniconda3:4.12.0

SHELL ["/bin/bash", "-ceuxo", "pipefail"]

ENV DEBIAN_FRONTEND=noninteractive

RUN conda install python=3.8.5 && conda clean -a -y
RUN conda install pytorch==1.11.0 torchvision==0.12.0 cudatoolkit=11.3 -c pytorch && conda clean -a -y

RUN apt-get update && apt install fonts-dejavu-core rsync gcc -y && apt-get clean


RUN <<EOF
git config --global http.postBuffer 1048576000
git clone https://github.com/sd-webui/stable-diffusion-webui.git stable-diffusion
cd stable-diffusion
git reset --hard 7623a5734740025d79b710f3744bff9276e1467b
conda env update --file environment.yaml -n base
conda clean -a -y
EOF


# Latent diffusion
RUN <<EOF
git clone --depth 1 https://github.com/Hafiidz/latent-diffusion.git
cd latent-diffusion
# hacks all the way down
mv ldm ldm_latent &&
sed -i -- 's/from ldm/from ldm_latent/g' *.py
# dont forget to update the yaml!!
EOF


# Note: don't update the sha of previous versions because the install will take forever
# instead, update the repo state in a later step
# ARG BRANCH=master  SHA=d0bb60a139d60e6c2b9be4e18e0e29a86aa5af59
ARG BRANCH=dev SHA=f585ab1923730339ea75d36b53ac1d5b6cbde500
RUN <<EOF
cd stable-diffusion
git fetch
git checkout ${BRANCH}
git reset --hard ${SHA}
conda env update --file environment.yaml -n base
conda clean -a -y
EOF

RUN pip install -U --no-cache-dir pyperclip

# add info
COPY . /docker/
RUN python /docker/info.py /stable-diffusion/frontend/frontend.py && chmod +x /docker/mount.sh

WORKDIR /stable-diffusion
ENV TRANSFORMERS_CACHE=/cache/transformers TORCH_HOME=/cache/torch PYTHONPATH="${PYTHONPATH}:/stable-diffusion" CLI_ARGS=""
EXPOSE 7860
# run, -u to not buffer stdout / stderr
CMD /docker/mount.sh && \
  python3 -u scripts/webui.py --outdir /output --ckpt /cache/models/model.ckpt --ldsr-dir /latent-diffusion ${CLI_ARGS}
  # STREAMLIT_SERVER_PORT=7860 python -m streamlit run scripts/webui_streamlit.py  --theme.base dark
