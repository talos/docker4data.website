<!-- 
.. title: Using Docker4Data
.. slug: docs
.. date: 2015-02-04 16:32:07 UTC-05:00
.. tags: 
.. link: 
.. description: 
.. type: text
-->

# Installing

Docker4data is as portable as Docker itself.  As the container is large, it is
recommended that you install in a place with a fast network, which may be
a cloud provider rather than locally.  There are instructions below for both
DigitalOcean and AWS, but any machine with Docker on AUFS (which should include
Boot2Docker on Mac, although this has not been tested) should support
Docker4Data.

## Locally

If you have Docker [installed locally][], you can use Docker4Data to explore
large datasets immediately on your machine!

  [installed locally]: http://docs.docker.com/installation/

You'll still need to pull many GB of data down, however.  If you're not on
a fast wired connection, this could take a long time.  You may find it's still
faster to use a cloud provider, as they can provide superior network
throughput.  There are two examples (DigitalOcean and AWS) below.

You'll also need at least ~18GB of free disk space, and you'll need to make
sure that you're using the AUFS storage driver.

Once Docker is running (on Mac, you will need to make sure to run
`boot2docker`) you can skip to Pull Down the Image.

## DigitalOcean

DigitalOcean is the easiest way to get started with a large dataset using
Docker4Data, as they supply an Ubuntu image pre-stocked with Docker using AUFS.

### Create the instance

Log into [DigitalOcean][], then select "Create Droplet".  Name your droplet.
You can choose the cheapest ($5/month) option.  Under "Select Image", choose
the "Applications" tab and select "Docker 1.4.1 on 14.04".  Create the droplet,
and wait about a minute for it to be provisioned.

  [DigitalOcean]: https://www.digitalocean.com

### Log in

You can either use the "Console Access" button or log in via root at the public
IP.

```
$ ssh root@my.ip.ad.rr
```

You're now ready to pull down the image (below).

## AWS

AWS is a great option to explore data, as billing is pro-rated and a free-tier
eligible instance should be sufficient to get started.  However, Docker doesn't
come pre-installed on Amazon's Ubuntu instance, so setup is more involved than
with DigitalOcean.

### Create the instance

First, create an Ubuntu instance. "Ubuntu Server 14.04 LTS (HVM), SSD Volume
Type - ami-9a562df2" and "t2.micro" should be fine.  You'll need to do "Step 3:
Configure Instance Details" in order to enable "Auto-assign public IP".  You'll
also need to up the amount of hard disk available on `dev/sda1` to 20GB in
"Step 4: Add Storage".

### Log in

Once the instance is created, click the "Connect" button in the AWS console.
This should supply your IP address to connect to.  The connection string should
look like this, except with your private key and IP:

```
ssh -i /path/to/private-key.pem ubuntu@my.ip.ad.dr
```

Once you're in, we need to make sure we have enough disk space.

```
df -h
```

If `/dev/xvda1` doesn't show at least 15G under "Avail", you'll need to
re-create this instance with more hard disk in "Step 4" above.

### Open port 8080

By default, we'll run the Docker4Data web frontend on port 8080.  You'll need
to allow inbound TCP traffic for this port.

The simplest way is to click on your instance, then the security group next to
"Security Groups".  Then click "Inbound", then "Edit", and "Add Rule".  Select
"Custom TCP Rule", and enter "8080" under "Port Range".  Choose "Anywhere" as
the source unless you know your IP and wish to limit access to it exclusively.

### Install Docker

It's not too difficult to get Docker running on Ubuntu.  You'll first need to
install AUFS, as without it Docker doesn't play nice with really big images.

```
$ sudo apt-get update
$ sudo apt-get -y install linux-image-extra-$(uname -r)
```

#### Easy way

If you're someone who trusts "curl it and sudo it".

```
$ curl -sSL https://get.docker.com/ubuntu/ | sudo sh
```

#### Slightly more complicated way

If you don't. ;)

```
$ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 36A1D7869245C8950F966E92D8576A8BA88D21E9
$ sudo sh -c "echo deb https://get.docker.com/ubuntu docker main > /etc/apt/sources.list.d/docker.list"
$ sudo apt-get update
$ sudo apt-get -y install lxc-docker
```

Make sure that Docker is installed properly.

```
$ sudo docker --version
Docker version 1.4.1, build 5bc2ff8
```

If that looks dandy, let's pull down the image!

# Pull down the image

This should take between 9 and 11 minutes on DigitalOcean, and a little less on
AWS.  Grab some coffee.  [Play Minesweeper](http://play-minesweeper.com/).

```
$ sudo docker pull thegovlab/docker4data-acris
```

# Play with the data

First, you'll need to launch the image.  This will run the web interface on
port 8080.  Change the number before the colon to change to a different port.

```
$ sudo docker run --name acris -d -p 8080:8080 thegovlab/docker4data-acris
```

You should now be able to navigate to the IP of your DigitalOcean or Amazon
instance and make SQL queries directly on the data there using a web interface.
If you're running locally, the web interface will be at
[http://localhost:8080](http://localhost:8080).

If you want to make queries directly in the commandline, you can drop into psql
directly.

```
$ sudo docker exec -i acris gosu postgres psql
```

