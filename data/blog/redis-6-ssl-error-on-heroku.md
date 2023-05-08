---
title: How to solve the SSL error for Redis 6 on Heroku?
date: '2022-02-03'
tags: ['redis', 'ssl', 'heroku', 'sidekiq', 'ruby']
draft: false
---

If you try to connect a Ruby on Rails app with a Heroku Redis add-on (excluding the Hobby Dev plan), there is a very high chance for you to get the error below:

```ruby
OpenSSL::SSL::SSLError: SSL_connect returned=1 errno=0 state=error:
certificate verify failed (self signed certificate in certificate chain)
```

## Reason

From version 6 and above, Redis requires using TLS to connect. However, Heroku does not use SSL internally. They terminate SSL at the router level and [forward](https://devcenter.heroku.com/articles/http-routing#routing) requests from there to your application via HTTP, which is safe as all these do happen behind Heroku’s firewall. Also, let’s face it, Heroku’s security measures –probably– are better than yours.

## Solution

To fix this, you will need to use `OpenSSL::SSL::VERIFY_NONE` for your Redis client.

```ruby
Redis.new(
  url: 'url',
  driver: :ruby,
  ssl_params: { verify_mode: OpenSSL::SSL::VERIFY_NONE }
)
```

If you do use Sidekiq, configuration should be done through the Sidekiq initializers:

```ruby
# config/initializers/sidekiq.rb

Sidekiq.configure_server do |config|
  config.redis = { ssl_params: { verify_mode: OpenSSL::SSL::VERIFY_NONE } }
end

Sidekiq.configure_client do |config|
  config.redis = { ssl_params: { verify_mode: OpenSSL::SSL::VERIFY_NONE } }
end
```

---

## Troubleshooting

### No error with Redis 6!?

#### Why?

Assuming you have double-checked the Redis version, the plan is probably on the Hobby Dev version, which does support both HTTP and HTTPS [connections](https://devcenter.heroku.com/articles/heroku-redis#create-a-new-instance).

#### Solution

For the Hobby Dev plan, you should see two environment variables set under the app’s “Config Vars” section. If you do plan to keep the add-on as Hobby Dev, no change is needed.

If you do plan to upgrade the add-on to Premium 0 or above, you need to use `VERIFY_NONE` as above.

### Links

- https://stackoverflow.com/questions/65834575/how-to-enable-tls-for-redis-6-on-sidekiq
