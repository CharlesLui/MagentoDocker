# ARG SSH_PUBLIC_KEY
# RUN mkdir -p /root/.ssh
# RUN echo "${SSH_PRIVATE_KEY}" > /root/.ssh/id_rsa
# RUN echo "${SSH_PUBLIC_KEY}" > /root/.ssh/id_rsa.pub
# RUN chmod 600 /root/.ssh/id_rsa
# RUN chmod 644 /root/.ssh/id_rsa.pub
# RUN touch /root/.ssh/known_hosts
# RUN ssh-keyscan git.corp.adobe.com >> /root/.ssh/known_hosts
ARG BASE
FROM $BASE AS build
ARG COMPOSER_AUTH
ARG BRANCH
ARG GIT_REPO
RUN apk update && apk add git bash
ENV COMPOSER_ALLOW_SUPERUSER=1
ENV COMPOSER_HOME=/var/composer
ENV COMPOSER_AUTH=$COMPOSER_AUTH

RUN git clone $GIT_REPO  -b $BRANCH  --depth 1 . && \
    rm -rf .git

RUN composer install --no-dev --no-interaction --prefer-dist

RUN vendor/bin/ece-tools patch

# workaround for the https://github.com/magento/magento2/issues/10041
#ADD https://raw.githubusercontent.com/magento/magento2/2.4.2/dev/tests/integration/testsuite/Magento/Setup/Console/Command/_files/config/dump_config.php /app/app/etc/config.php
ADD config.php /app/app/etc/config.php


RUN bin/magento module:enable -vvv --all
#RUN bin/magento module:disable Acidgreen_LayeredNavigation
RUN bin/magento module:disable Magento_TwoFactorAuth
RUN bin/magento module:disable -f Magento_AdvancedCheckout Magento_GiftCardAccount
RUN bin/magento module:disable Smile_ElasticsuiteCatalogGraphQl
RUN bin/magento module:disable Smile_ElasticsuiteIndices
#RUN bin/magento module:disable -f Smile_ElasticsuiteVirtualCategory
#RUN bin/magento module:disable -f Magento_Elasticsearch7
#RUN bin/magento module:disable -f Magento_ElasticsearchCatalogPermissions
#RUN bin/magento module:disable -f Magento_InventoryElasticsearch
#RUN bin/magento module:disable -f Magento_Elasticsearch6
#RUN bin/magento module:disable -f Magento_Elasticsearch
#RUN bin/magento module:disable -f Magento_AdvancedSearch

RUN bin/magento -vvv setup:di:compile
RUN composer dump-autoload --optimize

RUN bin/magento s:s:d en_US en_AU -vvv -f
RUN bin/magento module:status -vvv

RUN rm -rf $COMPOSER_HOME /app/dev /app/phpserver  /app/var/* /tmp/*
FROM $BASE

COPY --from=build /app /app
EXPOSE 9000

RUN apk add postfix
RUN echo 'relayhost = [postfix-relay.mail.svc.cluster.local]' >> /etc/postfix/main.cf

ADD start.sh /
RUN chmod +x /start.sh
CMD ["/start.sh"]
