FROM steveltn/https-portal:1
# 日本時間に設定する
RUN ln -sf  /usr/share/zoneinfo/Asia/Tokyo /etc/localtime
ADD ./nginx/conf.d/* /var/lib/nginx-conf/
