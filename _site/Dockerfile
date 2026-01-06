FROM ruby:3.3-bookworm

# Basics for building native gems + nice dev defaults
RUN apt-get update && apt-get install -y \
  build-essential \
  git \
  && rm -rf /var/lib/apt/lists/*

WORKDIR /srv/jekyll

# Configure bundler to install gems into a persisted volume path
ENV BUNDLE_PATH=/usr/local/bundle \
    BUNDLE_JOBS=4 \
    BUNDLE_RETRY=3

EXPOSE 4000 35729

CMD ["bash", "-lc", "bundle install && bundle exec jekyll serve --host 0.0.0.0 --livereload --incremental --force_polling"]