AWS_PROFILE ?= default
STACK_NAME ?= {{ cookiecutter.project_slug }}
S3_BUCKET_NAME ?= aws-sam-apps

BUILD_DIR = build

.PHONY: clean
clean:
	@rm -rf $(BUILD_DIR)/*
	@aws s3 rm \
		--profile $(AWS_PROFILE) \
		--recursive \
		s3://$(S3_BUCKET_NAME)/$(STACK_NAME)/*

.PHONY: build
build: $(BUILD_DIR)/template.yaml

$(BUILD_DIR)/template.yaml: template.sam.yaml src/requirements.txt src/*.py
	@sam build \
		--profile $(AWS_PROFILE) \
		--build-dir $(BUILD_DIR)

.PHONY: local-invoke
local-invoke: | build
	@sam local invoke \
		--profile $(AWS_PROFILE) \
		--template $(BUILD_DIR)/template.yaml \
		--event event.json

.PHONY: package
package: | build
	@sam package \
		--profile $(AWS_PROFILE) \
		--s3-bucket s3://$(S3_BUCKET_NAME)/$(STACK_NAME)/ \
		--template-file $(BUILD_DIR)/template.yaml \
		--output-template-file $(BUILD_DIR)/template-packaged.yaml

.PHONY: deploy
deploy: | package
	@sam deploy \
		--profile $(AWS_PROFILE) \
		--s3-bucket s3://$(S3_BUCKET_NAME)/$(STACK_NAME)/ \
		--template-file $(BUILD_DIR)/template.yaml \
		--stack-name $(STACK_NAME)

.PHONY: destroy
destroy: | clean
	@aws cloudformation delete-stack \
		--profile $(AWS_PROFILE) \
		--stack-name $(STACK_NAME)
