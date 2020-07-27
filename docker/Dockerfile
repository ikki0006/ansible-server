FROM ubuntu:18.04

RUN apt-get update && apt-get install -y openssh-server python
RUN mkdir /var/run/sshd
RUN echo 'root:tmppass' | chpasswd
RUN sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config

# SSH login fix. Otherwise user is kicked off after login
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd

ENV NOTVISIBLE "in users profile"
RUN echo "export VISIBLE=now" >> /etc/profile

# Create User
RUN apt-get install -y less vim sudo
RUN useradd -m admin
RUN echo 'admin:tmppass' | chpasswd
RUN usermod -aG sudo admin
RUN chsh -s /bin/bash admin

EXPOSE 22
CMD ["/sbin/init", "-D"]