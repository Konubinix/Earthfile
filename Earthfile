VERSION 0.6

alpine:
    FROM alpine
    # setup time-zone https://wiki.alpinelinux.org/wiki/Setting_the_timezone
    RUN apk add --update tzdata && cp /usr/share/zoneinfo/Europe/Paris /etc/localtime && echo "Europe/Paris" > /etc/timezone && apk del tzdata

alpine-python:
    FROM +alpine
    RUN apk add --update python3 py3-pip

debian:
   FROM debian:stable-slim
   DO +DEBIAN_NO_AUTO_INSTALL
   RUN rm /etc/localtime
   RUN ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime

pip-tools:
  FROM python:3.8-alpine
  DO +USE_USER
  RUN pip install pip-tools

SETUP_USER:
	COMMAND
    ARG uid=1000
    ARG username=sam
    ENV HOME=/home/$username
    RUN addgroup --gid $uid --system $username \
    	&& adduser --uid $uid --shell /bin/sh --disabled-password $username --ingroup $username \
		&& chown -R $username:$username $HOME
	ARG groups=audio video disk lp dialout users
	IF  [ -n "$groups" ]
		RUN for group in $groups ; do addgroup --system $group && adduser $username $group ; done
	END
    ENV PATH=$HOME/.local/bin:$PATH

AS_USER:
	COMMAND
    ARG uid=1000
    ARG username=sam
    ENV HOME=/home/$username
    USER $username
    WORKDIR $HOME

USE_USER:
	COMMAND
	ARG groups
    ARG uid=1000
    ARG username=sam
    DO +SETUP_USER --groups="$groups" --uid="${uid}" --username="${username}"
    DO +AS_USER --uid="${uid}" --username="${username}"

DEBIAN_NO_AUTO_INSTALL:
  COMMAND
  RUN echo 'APT::Install-Recommends "0";' > /etc/apt/apt.conf.d/01norecommend
  RUN echo 'APT::Install-Suggests "0";' >> /etc/apt/apt.conf.d/01norecommend

USER_WRITE_ENV:
    COMMAND
    ARG --required name
    RUN bash -c 'echo export ${name}="${!name}"' >> ${HOME}/.profile
