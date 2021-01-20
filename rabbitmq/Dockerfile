FROM daocloud.io/library/rabbitmq:3.7.7-management-alpine

COPY plugins/rabbitmq_delayed_message_exchange-3.8.0.ez /plugins

RUN rabbitmq-plugins enable rabbitmq_delayed_message_exchange