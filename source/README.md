Blog
=================
[![Build Status](https://travis-ci.org/szpssky/szpssky.github.com.svg?branch=blog-source)](https://travis-ci.org/szpssky/szpssky.github.com)

### Overview
This is a blog system builded by Travis CI.
### How to using Travis CI for Hexo
#### Preparation
Create ```travis.yml```
``` YAML
language: node_js
sudo: false
node_js:
  - "4"
cache:
  npm: ture
  directories:
    - node_modules
    - themes/tranquilpeak/node_modules
branches:
  only:
  - blog-source
  
```
Turn on the repository switch in the Travis.


#### Setp 1
Generating a new SSH key.
``` shell
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```
It will generate two files, ```ssh_key``` and ```ssh_key.pub```.

Copies the contents of the ssh_key.pub file to your clipboard and adding it to the Repository`s Deploy Keys.
#### Setp 2
Encryption private key.

Install Travis CI Command line client
``` shell
gem install travis
```
Login in travis
``` shell
travis login --auto
```
Encrypted ssh_key
``` shell
travis encrypt-file id_rsa --add
```
It will generate ```ssh_key.enc``` file and automatic write some config into ```.travis.yml```.

Modify ```.travis.yml``` file
``` YAML
- openssl aes-256-cbc -K $encrypted_*****_key -iv $encrypted_*****_iv
  -in ssh_key.enc -out ~/.ssh/id_rsa -d
```
#### Setp 3
It will make building systems to establish a default ssh connection

Create ```ssh_config``` file and modify ```.travis.yml```
``` 
Host github.com
  User git
  StrictHostKeyChecking no
  IdentityFile ~/.ssh/id_rsa
  IdentitiesOnly yes
```
``` YAML
- chmod 600 ~/.ssh/id_rsa
- eval $(ssh-agent)
- ssh-add ~/.ssh/id_rsa
- cp ssh_config ~/.ssh/config
```
#### Setp 4
Travis Full configuration.
``` YAML
language: node_js

sudo: false
node_js:
  - "4"

cache:
  npm: ture
  directories:
    - node_modules
branches:
  only:
  - blog-source

before_install:
- openssl aes-256-cbc -K $encrypted_*****_key -iv $encrypted_*****_iv
  -in ssh_key.enc -out ~/.ssh/id_rsa -d
- chmod 600 ~/.ssh/id_rsa
- eval $(ssh-agent)
- ssh-add ~/.ssh/id_rsa
- cp ssh_config ~/.ssh/config
- git config --global user.name "yourname"
- git config --global user.email "your_email@example.com"

install:
- npm install -g hexo-cli
- npm install
- npm install hexo-generator-sitemap --save
- npm install hexo-generator-baidu-sitemap --save

script:
- hexo cl
- hexo g
- hexo d
```
#### Finally
Commit and push to the repository.Enjoy It!
