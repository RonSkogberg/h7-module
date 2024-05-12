# -*- mode: ruby -*-
# vi: set ft=ruby :
# Copyright 2014-2023 Tero Karvinen http://TeroKarvinen.com
# Modified by Ron Skogberg

Vagrant.configure("2") do |config|

	config.vm.box = "debian/bullseye64"
	config.vm.define "heikkihelppari" do |heikkihelppari|
		heikkihelppari.vm.hostname = "heikkihelppari"
		heikkihelppari.vm.network "private_network", ip: "192.168.88.101"
	end
	config.vm.define "tiinatoimari", primary: true do |tiinatoimari|
		tiinatoimari.vm.hostname = "tiinatoimari"
		tiinatoimari.vm.network "private_network", ip: "192.168.88.102"
	end
	config.vm.define "raimoraksaukko", primary: true do |raimoraksaukko|
		raimoraksaukko.vm.hostname = "raimoraksaukko"
		raimoraksaukko.vm.network "private_network", ip: "192.168.88.103"
	end
end