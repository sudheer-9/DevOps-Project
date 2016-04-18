# DevOps-Project

#create a module nginx with manifestes,files and tests dir 

##In manifestes create a file for installing nginx and start service 
#vi modules/nginx/manifests/nginx.pp 
class nginx {
  package { 'nginx':
    ensure => installed,
  }

  service { 'nginx':
    ensure => running,
    require => Package['nginx'],
  }

  file { '/etc/nginx/sites-enabled/default':
    source => 'puppet:///modules/nginx/virtualserver.conf',
    notify => Service['nginx'],
  }
  file { '/var/www/test/':
soruce => 'puppet:///moduels/nginx/hello.hmtl'
}
}
###create file for html virtual servers 
#vi modules/nginx/files/virtualserver.conf

server {
  listen 80;
  listen   443 ssl;
  root /var/www/test;
  ssl_certificate /etc/ssl/certs/www.test.com.pem;
  ssl_certificate_key /etc/ssl/private/www.test.com.key;
  server_name test.com;
}

###create file for html 
#vi modules/nginx/files/hello.html 
<html>
<head>
<title>Hello World</title>
</head>
<body>
<h1>Hello World!</h1>
</body>
</html>


# write a rpesc script in spec dir
#vi modules/nginx/spec/nginx_spec.rb

require 'spec_helper'
describe 'nginx' do 
  let :params do
    {
      :nginx_vhosts          => { 'test.local' => { 'www_root' => '/var/www/test' } },
      :nginx_vhosts_defaults => { 'listen_options' => 'default_server' },
      :nginx_locations       => { 'test.local' => { 'vhost' => 'test.local', 'www_root' => '/var/www/test'} },
      }
  end

  describe "with defaults" do
    it { is_expected.to compile.with_all_deps }
    it { is_expected.to contain_class('nginx') }
    it { is_expected.to contain_anchor('nginx::begin') }
    it { is_expected.to contain_nginx__package.that_requires('Anchor[nginx::begin]') }
    it { is_expected.to contain_nginx__config.that_requires('Class[nginx::package]') }
    it { is_expected.to contain_nginx__service.that_subscribes_to('Anchor[nginx::begin]') }
    it { is_expected.to contain_nginx__service.that_subscribes_to('Class[nginx::package]') }
    it { is_expected.to contain_nginx__service.that_subscribes_to('Class[nginx::config]') }
    it { is_expected.to contain_anchor('nginx::end').that_requires('Class[nginx::service]') }
    it { is_expected.to contain_nginx__resource__vhost("test.local") }
    it { is_expected.to contain_nginx__resource__vhost("test.local").with_listen_options('default_server') }
    it { is_expected.to contain_nginx__resource__location("test.local") }
    end
end
