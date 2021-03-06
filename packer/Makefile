ENCRYPT_CONFIG := files/encrypt.hcl

# if AMI_PREFIX is changed, new value must be specified in Terraform code
# 	 using ami_prefix parameter for consul-demo-cluster module
AMI_PREFIX="consul-demo"
AMI_OWNER=self
CONSUL_VER="1.7.3"
BUILD_DATE := $(shell date +%Y%m%d-%H%M)

DEFAULT:
	@echo Run 'make aws' to build appropriate images

.PHONY: aws
aws: aws_consul_server aws_client_mongodb aws_client_product aws_client_webclient aws_client_listing

.PHONY: aws_consul_base
aws_consul_base: .built.aws_consul_base

.PHONY: aws_consul_client
aws_consul_client: aws_consul_base .built.aws_consul_client

.PHONY: aws_consul_server
aws_consul_server: aws_consul_client .built.aws_consul_server

.PHONY: aws_client_listing
aws_client_listing: aws_consul_client .built.aws_client_listing

.PHONY: aws_client_mongodb
aws_client_mongodb: aws_consul_client .built.aws_client_mongodb

.PHONY: aws_client_product
aws_client_product: aws_consul_client .built.aws_client_product

.PHONY: aws_client_webclient
aws_client_webclient: aws_consul_client .built.aws_client_webclient

.built.aws_consul_base: consul_base.json files/install_base.sh files/install_consul.sh
	BUILD_DATE=$(BUILD_DATE) AMI_OWNER=$(AMI_OWNER) CONSUL_VER=$(CONSUL_VER) AMI_PREFIX=$(AMI_PREFIX) packer build -only amazon-ebs consul_base.json
	touch .built.aws_consul_base

.built.aws_consul_client: consul_client.json files/auto-join-aws.hcl.tmpl files/update-consul-join-config files/client.hcl
	BUILD_DATE=$(BUILD_DATE) AMI_OWNER=$(AMI_OWNER) CONSUL_VER=$(CONSUL_VER) AMI_PREFIX=$(AMI_PREFIX) packer build -only amazon-ebs consul_client.json
	touch .built.aws_consul_client

.built.aws_consul_server: consul_server.json files/server.hcl
	BUILD_DATE=$(BUILD_DATE) AMI_OWNER=$(AMI_OWNER) CONSUL_VER=$(CONSUL_VER) AMI_PREFIX=$(AMI_PREFIX) packer build -only amazon-ebs consul_server.json
	touch .built.aws_consul_server

.built.aws_client_listing: client_listing.json files/install_node.sh services/listing.hcl files/listing.service
	BUILD_DATE=$(BUILD_DATE) AMI_OWNER=$(AMI_OWNER) CONSUL_VER=$(CONSUL_VER) AMI_PREFIX=$(AMI_PREFIX) packer build -only amazon-ebs client_listing.json
	touch .built.aws_client_listing

.built.aws_client_mongodb: client_mongodb.json services/mongodb.hcl files/mongod.conf
	BUILD_DATE=$(BUILD_DATE) AMI_OWNER=$(AMI_OWNER) CONSUL_VER=$(CONSUL_VER) AMI_PREFIX=$(AMI_PREFIX) packer build -only amazon-ebs client_mongodb.json
	touch .built.aws_client_mongodb

.built.aws_client_product: client_product.json services/product.hcl files/product.service
	BUILD_DATE=$(BUILD_DATE) AMI_OWNER=$(AMI_OWNER) CONSUL_VER=$(CONSUL_VER) AMI_PREFIX=$(AMI_PREFIX) packer build -only amazon-ebs client_product.json
	touch .built.aws_client_product

.built.aws_client_webclient: client_webclient.json services/web_client.hcl files/web_client.service
	BUILD_DATE=$(BUILD_DATE) AMI_OWNER=$(AMI_OWNER) CONSUL_VER=$(CONSUL_VER) AMI_PREFIX=$(AMI_PREFIX) packer build -only amazon-ebs client_webclient.json
	touch .built.aws_client_webclient

$(ENCRYPT_CONFIG):
	$(if $(shell PATH=$(PATH) which consul),,$(error "No consul in PATH"))
	@echo Making encryption key
	echo "encrypt = $$(consul keygen)" > $(ENCRYPT_CONFIG)
