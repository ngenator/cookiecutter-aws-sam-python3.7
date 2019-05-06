export AWS_PROFILE ?= default
export AWS_REGION ?= us-east-1

STACK_NAME ?= {{ cookiecutter.project_slug }}
S3_BUCKET_NAME ?= aws-sam-apps

BUILD = ./build

TEMPLATE = template.yaml
PACKAGED = packaged.yaml

.PHONY: clean
clean:
	@rm -rf $(BUILD)

.PHONY: build
build: $(BUILD)

$(BUILD)/$(TEMPLATE): $(TEMPLATE) src/*.py
	@sam build \
		--build-dir $(BUILD) \
		--use-container \
		--manifest requirements.txt

.PHONY: local-invoke
local-invoke: | build
	@sam local invoke \
		--template $(BUILD)/$(TEMPLATE) \
		--event event.json

.PHONY: package
package: | build
	@sam package \
		--s3-bucket s3://$(S3_BUCKET_NAME)/$(STACK_NAME)/ \
		--template-file $(BUILD)/$(TEMPLATE) \
		--output-template-file $(BUILD)/$(PACKAGED)

.PHONY: deploy
deploy: | package
	@sam deploy \
		--s3-bucket s3://$(S3_BUCKET_NAME)/$(STACK_NAME)/ \
		--template-file $(BUILD)/$(PACKAGED) \
		--stack-name $(STACK_NAME)

.PHONY: destroy
destroy: | clean
	@aws cloudformation delete-stack \
		--stack-name $(STACK_NAME)
