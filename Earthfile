alpine:
	FROM alpine
    # setup time-zone https://wiki.alpinelinux.org/wiki/Setting_the_timezone
    RUN apk add --update tzdata && cp /usr/share/zoneinfo/Europe/Paris /etc/localtime && echo "Europe/Paris" > /etc/timezone && apk del tzdata

alpine-python:
	FROM +alpine
    RUN apk add --update python3 py3-pip

SETUP_USER:
	COMMAND
    ARG uid=1000
    ARG username=sam
    ENV HOME=/home/$username
    RUN addgroup --gid $uid --system $username \
    	&& adduser --uid $uid --system $username --ingroup $username \
		&& chown -R $username:$username $HOME
	ARG groups=audio video disk lp dialout users
	IF  [ -n "$group" ]
		RUN for group in $groups ; do addgroup --system $username $group ; done
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
    DO +SETUP_USER --groups="$groups"
    DO +AS_USER
