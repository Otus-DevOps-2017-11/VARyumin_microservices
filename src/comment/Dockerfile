FROM ruby:2.2-alpine
RUN apk add --update alpine-sdk
ENV APP_HOME /app
RUN mkdir $APP_HOME
WORKDIR $APP_HOME
COPY Gemfile* $APP_HOME/
RUN bundle install
COPY . $APP_HOME
ENV COMMENT_DATABASE_HOST comment_db
ENV COMMENT_DATABASE comments
CMD ["puma"]
