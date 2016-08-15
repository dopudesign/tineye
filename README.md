# tineye
A ruby class for interacting with the TinEye API

```
require 'httparty'
require 'securerandom'
require 'time'
require 'uri'

class Tineye
  include HTTParty

  API_URL = 'https://api.tineye.com/rest/search/'.freeze
  PUBLIC_KEY = ENV['TINEYE_PUBLIC']
  PRIVATE_KEY = ENV['TINEYE_PRIVATE']

  def self.query(request_url)
    request_date = Time.now.to_i
    request_nonce = nonce
    request_signature = signature(request_date, request_nonce, request_url)

    get(API_URL,
        query: {
          api_key: PUBLIC_KEY,
          image_url: request_url,
          date: request_date,
          nonce: request_nonce,
          api_sig: request_signature })
  end

  def self.signature(request_date, request_nonce, request_url)
    params = { image_url: request_url }
    request_params = URI.encode_www_form(params)
    to_sign = "#{PRIVATE_KEY}GET#{request_date}#{request_nonce}#{API_URL}#{request_params}"
    OpenSSL::HMAC.hexdigest(OpenSSL::Digest.new('sha1'), PRIVATE_KEY, to_sign) if PRIVATE_KEY
  end

  def self.nonce
    @nonce = SecureRandom.hex(16)
  end
end
```
