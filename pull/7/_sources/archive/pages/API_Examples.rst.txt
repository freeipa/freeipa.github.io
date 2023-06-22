Some useful links:

-  Net::IPA

| ``Perl 5 interface of the FreeIPA JSON-RPC API``
| ``This library is going to be deprecaded and replaced by Net::FreeIPA.``
| ```https://github.com/NickCis/perl-Net-IPA`` <https://github.com/NickCis/perl-Net-IPA>`__

-  p5-net-freeipa

| ``Perl5 NET::FreeIPA (FreeIPA 4.2+ JSON API)``
| ```https://github.com/stdweird/p5-net-freeipa`` <https://github.com/stdweird/p5-net-freeipa>`__

-  Blog of ALEXANDER BOKOVOY

| ```https://vda.li/en/posts/2016/11/02/FreeIPA-JSON-RPC-API-article/`` <https://vda.li/en/posts/2016/11/02/FreeIPA-JSON-RPC-API-article/>`__
| ```https://vda.li/en/docs/freeipa-management-in-a-nutshell/`` <https://vda.li/en/docs/freeipa-management-in-a-nutshell/>`__
| ```https://vda.li/en/posts/2015/05/28/talking-to-freeipa-api-with-sessions/`` <https://vda.li/en/posts/2015/05/28/talking-to-freeipa-api-with-sessions/>`__

-  Red Hat article on the API

`` ``\ ```https://access.redhat.com/articles/2728021`` <https://access.redhat.com/articles/2728021>`__

-  attachment to this post shows how simple user/password auth works:

```https://www.redhat.com/archives/freeipa-users/2015-November/msg00132.html`` <https://www.redhat.com/archives/freeipa-users/2015-November/msg00132.html>`__

-  python-freeipa-json

| ``Tiny/basic module for communicating with the FreeIPA API without having to install their entire toolchain``
| ```https://github.com/nordnet/python-freeipa-json`` <https://github.com/nordnet/python-freeipa-json>`__

Example API perl script

| ``#!/usr/bin/perl``
| ``#``
| ``# Created by GG -- 20180306``
| ``# Version 1.0``
| ``# GPLv3``
| ``# this script adds a new entry to IPA DNS and creates a host object with otp for registration``
| ``# for our automatic deployments, after this the machine gets created in vmware, kickstarted and provisioned``
| `` ``
| ``use strict;``
| ``use REST::Client;``
| ``use JSON;``
| ``use ``\ ```Data::Dumper`` <Data::Dumper>`__\ ``;``
| `` ``
| ``# IPA server data``
| ``my $ipaserver="``\ ```https://ipa.yourdomain.com`` <https://ipa.yourdomain.com>`__\ ``";``
| ``my $ipausername="ipauser";``
| ``my $ipapassword="ipapassword";``
| `` ``
| ``## parameters for the entries to create``
| ``my $newhostname="newhost-01";``
| ``my $newdomain="yourdomain.com";``
| ``my $newip="192.168.1.1";``
| ``# We use a one time password in our scripts to register machines on firstboot``
| ``my $newotp="OneTimePassword";``
| ``## end parameters``
| `` ``
| ``create_ipa_entries($newhostname, $newdomain, $newip, $newotp, $ipaserver, $ipausername, $ipapassword);``
| `` ``
| ``sub create_ipa_entries {``
| `` ``
| ``  my ($hostname,$domain,$ip,$otp,$server,$username,$password) = @_;``
| `` ``
| ``  # this is the apiversion of the IPA server``
| ``  # this should only be changed if we are sure future versions support this syntax``
| ``  my $apiversion="2.213";``
| `` ``
| ``  my $fqdn=$hostname . "." . $domain;``
| ``  my $uri = URI->new($server);``
| `` ``
| ``  my $authheader= {``
| ``  'Accept' => 'text/plain',``
| ``  'referer' => $server . "/ipa",``
| ``  'Content-Type' => 'application/x-www-form-urlencoded',``
| ``  };``
| `` ``
| ``  my $jsonheader= {``
| ``  'Accept' => 'application/json',``
| ``  'referer' => $server . "/ipa",``
| ``  'Content-Type' => 'application/json',``
| ``  };``
| `` ``
| ``  my $authbody = {``
| ``  'user' => $username,``
| ``  'password' => $password,``
| ``  };``
| `` ``
| ``  # see FreeIPA api dnsrecord_add``
| ``  my $dnsadd = {``
| ``      "method" => "dnsrecord_add",``
| ``      "params" => [``
| ``          [``
| ``              $domain,``
| ``              {``
| ``                  "__dns_name__" => $hostname``
| ``              }``
| ``          ],``
| ``          {``
| ``              "a_extra_create_reverse" => "true",``
| ``              "a_part_ip_address" => $ip,``
| ``              "version" => $apiversion``
| ``          }``
| ``      ],``
| ``      "id" => 0``
| ``  };``
| `` ``
| ``  # see FreeIPA api host_add``
| ``  my $hostadd = {``
| ``      "method" => "host_add",``
| ``      "params" => [``
| ``          [``
| ``              $fqdn``
| ``          ],``
| ``          {``
| ``              "userpassword" => $otp,``
| ``          "version" => $apiversion``
| ``          }``
| ``      ],``
| ``      "id" => 0``
| ``  };``
| `` ``
| ``  # create useragent with cookie support``
| ``  my $ua = LWP::UserAgent->new( cookie_jar => {} );``
| ``  my $client = REST::Client->new( { useragent => $ua } );``
| `` ``
| ``  # login with user / password and get a session cookie``
| ``  my $params = $client->buildQuery($authbody);``
| ``  $client->setHost($server);``
| ``  $client->POST("/ipa/session/login_password", substr($params, 1), $authheader);``
| ``  #print Dumper $client->responseHeader('Set-Cookie');``
| ``  if ($client->responseContent()) {``
| ``    print "IPA authentication error\n";``
| ``    # the output underneath is html formatted but I don't want to parse it here``
| ``    # so quick and dirty raw output``
| ``    print Dumper $client->responseContent();``
| ``    exit 1;``
| ``  }``
| ``  else {``
| ``    print "IPA auth ok\n";``
| ``  }``
| `` ``
| ``  # add host to DNS``
| ``  $client->POST("/ipa/session/json", encode_json($dnsadd), $jsonheader);``
| ``  #print Dumper $client->responseContent();``
| ``  my $result = decode_json($client->responseContent());``
| ``  if ($result->{"error"}{"code"}) {``
| ``    print "ERROR: ". $result->{"error"}{"code"} . " " . $result->{"error"}{"name"} . "\n";``
| ``    print $result->{"error"}{"message"} . "\n";``
| ``    exit 1;``
| ``  }``
| ``  else {``
| ``    print "dns entry " . $fqdn . " with ip address " . $ip . " created in IPA\n";``
| ``  }``
| `` ``
| ``  # create host object in IPA and set otp``
| ``  $client->POST("/ipa/session/json", encode_json($hostadd), $jsonheader);``
| ``  #print Dumper $client->responseContent();``
| ``  my $result = decode_json($client->responseContent());``
| ``  if ($result->{"error"}{"code"}) {``
| ``    print "ERROR: ". $result->{"error"}{"code"} . " " . $result->{"error"}{"name"} . "\n";``
| ``    print $result->{"error"}{"message"} . "\n";``
| ``    exit 1;``
| ``  }``
| ``  else {``
| ``    print "host object " . $fqdn . " with otp " . $otp . " created in IPA\n";``
| ``  }``
| ``}``
