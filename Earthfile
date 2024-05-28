VERSION 0.8

# https://www.debian.org/releases/
ARG --global DEBIAN_VERSION=12
ARG --global DEBIAN_MIN_VERSION=5


alpine:
    FROM alpine:3.18.4
    DO +ALPINE_TZ_FR
    ARG extra_packages
    IF test -n "${extra_packages}"
        RUN apk add ${extra_packages}
    END

ALPINE_TZ_FR:
    FUNCTION
    # setup time-zone https://wiki.alpinelinux.org/wiki/Setting_the_timezone
    RUN apk add --update tzdata && cp /usr/share/zoneinfo/Europe/Paris /etc/localtime && echo "Europe/Paris" > /etc/timezone && apk del tzdata

alpine-tz:
    FROM +alpine
    RUN mkdir /output
    RUN cp /etc/localtime /etc/timezone /output/
    SAVE ARTIFACT /output

alpine-python:
    ARG extra_packages
    FROM +alpine --extra_packages="python3 py3-pip ${extra_packages}"

PYTHON_VENV:
    FUNCTION
    ARG --required base
    RUN python3 -m venv --system-site-packages ${base}/venv
    ENV PATH=${base}/venv/bin:${PATH}
    ARG packages
    IF test -n "${packages}"
        RUN ${base}/venv/bin/python -m pip install --upgrade ${packages}
    END

alpine-python-user-venv:
    ARG dev=false
    ARG extra_packages
    FROM +alpine-python --dev=${dev} --extra_packages="${extra_packages}"
	ARG groups
    DO +USE_USER --groups="${groups}"
    ARG packages
    ARG workdir=/app
    WORKDIR ${workdir}
    DO +PYTHON_VENV --base=${workdir} --packages=${packages}

DEBIAN_APT_CLEANUP:
    FUNCTION
    RUN apt-get clean && \
        rm -rf /var/lib/apt/lists/*

debian:
    FROM debian:${DEBIAN_VERSION}.${DEBIAN_MIN_VERSION}-slim
    DO +DEBIAN_NO_AUTO_INSTALL
    DO +DEBIAN_TZ_FR
    ARG extra_packages
    IF test -n "${extra_packages}"
        RUN apt-get update && apt-get install --yes ${extra_packages}
        DO +DEBIAN_APT_CLEANUP
    END

debian-python-user-venv:
    ARG extra_packages
    FROM +debian --extra_packages="python3-venv ${extra_packages}"
	ARG groups
    DO +USE_USER --groups="${groups}"
    ARG packages
    ARG workdir=/app
    WORKDIR ${workdir}
    DO +PYTHON_VENV --base=${workdir} --packages=${packages}

DEBIAN_TZ_FR:
   FUNCTION
   RUN rm /etc/localtime
   RUN ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime

debian-europe-paris:
    FROM debian:${DEBIAN_VERSION}.${DEBIAN_MIN_VERSION}-slim
    SAVE ARTIFACT /usr/share/zoneinfo/Europe/Paris localtime

distroless-python3-debian:
    FROM gcr.io/distroless/python3-debian${DEBIAN_VERSION}
    COPY +debian-europe-paris/localtime /etc/localtime

distroless-debian:
    FROM gcr.io/distroless/static-debian${DEBIAN_VERSION}
    COPY +debian-europe-paris/localtime /etc/localtime

pip-tools:
  FROM python:3.8-alpine
  DO +USE_USER
  RUN pip install pip-tools

SETUP_USER:
	FUNCTION
    ARG uid=1000
    ARG username=sam
    ARG sudoer=n
    ARG shell=/bin/sh
    ENV HOME=/home/$username
    RUN addgroup --gid $uid --system $username \
    	&& adduser --uid $uid --shell ${shell} --disabled-password $username --ingroup $username \
		&& chown -R $username:$username $HOME
    # audio video disk lp dialout users
    IF test "${sudoer}" = "y"
        RUN mkdir -p /etc/sudoers.d && echo "${username} ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/username
    END
	ARG groups
	IF  [ -n "$groups" ]
		RUN for group in $groups ; do { grep -q -E "^${group}:" /etc/group || addgroup --system $group ; } && adduser $username $group ; done
	END
    ENV PATH=$HOME/.local/bin:$PATH

AS_USER:
	FUNCTION
    ARG uid=1000
    ARG username=sam
    ENV HOME=/home/$username
    USER $username
    WORKDIR $HOME

USE_USER:
	FUNCTION
	ARG groups
    ARG uid=1000
    ARG sudoer=n
    ARG username=sam
    DO +SETUP_USER --sudoer="${sudoer}" --groups="$groups" --uid="${uid}" --username="${username}"
    DO +AS_USER --uid="${uid}" --username="${username}"

DEBIAN_NO_AUTO_INSTALL:
  FUNCTION
  RUN echo 'APT::Install-Recommends "0";' > /etc/apt/apt.conf.d/01norecommend
  RUN echo 'APT::Install-Suggests "0";' >> /etc/apt/apt.conf.d/01norecommend

USER_WRITE_ENV:
    FUNCTION
    ARG --required name
    RUN bash -c 'echo export ${name}="${!name}"' >> ${HOME}/.profile
