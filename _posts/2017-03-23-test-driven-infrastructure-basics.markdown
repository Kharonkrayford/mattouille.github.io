---
title: "Test Driven Infrastructure Basics"
layout: post
date: 2017-03-23 19:15
headerImage: false
tag:
- inspec
star: true
category: blog
author: mattouille
description: My take on TDI with InSpec
---

Today I'm going to go over the basics of Test Driven Infrastructure, what it means, how to do it, when it applies, and why. In this tutorial I'm going to use Chef, but you can use whatever you want.

**Test Driven Infrastructure** (TDI) is a term that's been somewhat borrowed from the software engineering concept of **Test Driven Development** (TDD). While TDD is somewhat questioned in certain facets, everybody can agree that automated testing is the best way forward. TDI basically implies that we build the tests before we build out infrastructure. If this sounds a little shocking, let me explain:

Say I want to build a brand new server to host my blog and it's database. We know we have some initial requirements which can fall under compliance (Securing SSH, establishing firewall rules, etc...). We also know that I'll need Apache, PHP, and mySQL. Generally, we already know what to test before we build it as we've proven.

* Apache2 is installed & running as a service
* Apache2 listens on port 80
* mySQL is installed & running as a service
* mysqld is listening on port 3306/tcp via localhost

That wasn't too difficult. How do we turn that into tests? You're about to find out that it's almost as easy as speaking English.

<div class="alert alert-info"><i>Why didn't I mentioned I'm using InSpec before? Most test frameworks are mostly the same. This is just about the basics.</i></div>

Depending on what Configuration Management software you use, you'll have to configure your resource to be tested differently. Here's my .kitchen.yml file

{% highlight yaml %}
---
driver:
  name: vagrant

provisioner:
  name: chef_zero
  # You may wish to disable always updating cookbooks in CI or other testing environments.
  # For example:
  #   always_update_cookbooks: <%= !ENV['CI'] %>
  always_update_cookbooks: true

verifier:
  name: inspec

platforms:
  - name: ubuntu-16.04

suites:
  - name: default
    run_list:
      - recipe[mattouille::default]
    verifier:
      inspec_tests:
        - test/smoke/default
    attributes:
{% endhighlight %}

Our tests will be located relative to the .kitchen.yml file. Notice we specified `test/smoke/default`. It's labeled `default` because it's in reference to default.rb. This helps us stay organized. Next create `default_test.rb` in the `default` folder you just created.

*Note* Every test document should be formatted with `*_test.rb`

InSpec is nice and simple. You can be as simple or as complex as you want. Here's a <a href="http://inspec.io/docs/">copy of the documentation</a> in case you were wondering where I get all this magical goodness.

Let's start out by defining our Apache test. Remember what we wrote down earlier:

* Apache2 is installed & running as a service
* Apache2 listens on port 80

{% highlight ruby %}
describe package 'apache2' do
	it { should be_installed }
end

describe service(apache.service) do
	it { should be_enabled }
	it { should be_running }
end

describe port(80) do
	it { should be_listening }
	its('processes') { should include 'apache2' }
end
{% endhighlight %}

#### Explanation

InSpec (and other test frameworks) run agentless (from the host) after all the configuration magic has happened. First we check to make sure the package is installed. We can assume by this point that Apache *should* be up and running rather painlessly. It is empty after all, with just the default start page. At the same time we check to make sure that the service is enabled. InSpec knows how to interact with SystemD on it's own but has built in resources to check for SystemD, UpStart, etc... if you need them. Next, we make sure that the Apache2 process is the one listening on port 80. Now we're all set!

Lastly we'll tackle mySQL. Here's what we wrote down earlier:

* mySQL is installed & running as a service
* mysqld is listening on port 3306/tcp via localhost

{% highlight ruby %}
# --- Apache Stuff ---

describe package 'mysql-server' do
	it { should be_installed }
end

describe service(mysql.service) do
	it { should be_enabled }
	it { should be_running }
end

describe port(3306) do
	it { should be_listening }
	its('protocols') { should include 'tcp' }
	its('processes') { should include 'mysqld' }
	its('addresses') { should include '127.0.0.1'}
end
{% endhighlight %}

#### Explanation

MySQL gets a little trickier because you install the mysql-server package, there is a mysql service, however, mysqld is what handles all the interaction with data. First we check to see that the package is installed. Then we check to see that the service is enabled and running. Lastly we make sure that 3306 is open, listening for TCP requests via mysqld on 127.0.0.1.

<div class="alert alert-warning">If you go looking for the wrong process listening on a port, InSpec will actually tell you what process is listening. You should obviously verify that this is correct, but it definitely aids in the education process.</div>

#### Profiles

Each framework has their own grouping mechanisms. InSpec calls theirs profiles. Review the changes I've made to our script below:

{% highlight ruby %}
control 'Apache Daemon' do
	impact 1.0
	title 'Server: Configure daemon and service port'
	desc '
		Enable the service and specify the port used.
	'

	describe package 'apache2' do
		it { should be_installed }
	end

	describe service(apache.service) do
		it { should be_enabled }
		it { should be_running }
	end

	describe port(80) do
		it { should be_listening }
		its('processes') { should include 'apache2' }
	end
end

control 'MySQL Daemon' do
	impact 1.0
	title 'Server: Configure daemon and service port'
	desc '
		Enable the service and specify the port and address used.
	'

	describe package 'mysql-server' do
		it { should be_installed }
	end

	describe service(mysql.service) do
		it { should be_enabled }
		it { should be_running }
	end

	describe port(3306) do
		it { should be_listening }
		its('protocols') { should include 'tcp' }
		its('processes') { should include 'mysqld' }
		its('addresses') { should include '127.0.0.1'}
	end
end
{% endhighlight %}

I've added some additional information to cover the impact, group each test, and provide a brief description. At this point we could go ahead and write our configuration code. Once we're done we can run our tests and make sure everything lined up. Let's see what happens when I run `kitchen verify` on my code:

{% highlight shell %}
-----> Verifying <default-ubuntu-1604>...
       Loaded  

Target:  ssh://vagrant@127.0.0.1:2222

  ✔  Apache Daemon: Server: Configure daemon and service port
     ✔  System Package apache2 should be installed
     ✔  Service apache2 should be enabled
     ✔  Service apache2 should be running
     ✔  Port 80 should be listening
     ✔  Port 80 processes should include "apache2"
  ✔  MySQL Daemon: Server: Configure daemon and service port
     ✔  System Package mysql-server should be installed
     ✔  Service mysql should be enabled
     ✔  Service mysql should be running
     ✔  Port 3306 should be listening
     ✔  Port 3306 protocols should include "tcp"
     ✔  Port 3306 processes should include "mysqld"
     ✔  Port 3306 addresses should include "127.0.0.1"

Profile Summary: 2 successful, 0 failures, 0 skipped
Test Summary: 12 successful, 0 failures, 0 skipped
       Finished verifying <default-ubuntu-1604> (0m3.64s).
-----> Kitchen is finished. (0m4.66s)
{% endhighlight %}

You probably can't see it very well so I've highlighted the lines, but there's checks out to the left of each test and profile. This means they passed! There's obviously a lot more to discuss about TDI as well as testing that automated deployments occur correctly but that's for a follow-on article.

#### Where does TDI apply?

Well, this is kind of up to you. In terms of TDD, some people say you should build the tests first and the applications second. Others say you should be able to take an objective look at your infrastructure and understand the root of what it's trying to accomplish and test for results. There's arguments for both, but ultimately you and your organization need to make that decision. Regardless, deploying without testing is foolish.

#### Where can TDI be used?

TDI is really part of the CI/CD process. You can make this an integral part of your build process using Jenkins or GoCD, or you can test locally.

Thanks for reading!
