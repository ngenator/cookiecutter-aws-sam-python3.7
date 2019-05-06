export AWS_PROFILE ?= default
export AWS_REGION ?= us-east-1

STACK_NAME ?= {{ cookiecutter.project_slug }}
S3_BUCKET_NAME ?= aws-sam-apps

BUILD_DIR = ./build

TEMPLATE = template.yaml
PACKAGED = packaged.yaml

.PHONY: clean
clean:
	@rm -rf $(BUILD_DIR)

.PHONY: build
build: $(BUILD_DIR)/$(TEMPLATE)

$(BUILD_DIR):
	@mkdir -p $@
$(BUILD_DIR)/$(TEMPLATE): $(TEMPLATE) requirements.txt src/*.py
	@sam build \
		--build-dir $(BUILD_DIR) \
		--use-container \
		--manifest requirements.txt

.PHONY: local-invoke
local-invoke: | build
	@sam local invoke \
		--template $(BUILD_DIR)/$(TEMPLATE) \
		--event event.json

.PHONY: package
package: | build
	@sam package \
		--s3-bucket s3://$(S3_BUCKET_NAME)/$(STACK_NAME)/ \
		--template-file $(BUILD_DIR)/$(TEMPLATE) \
		--output-template-file $(BUILD_DIR)/$(PACKAGED)

.PHONY: deploy
deploy: | package
	@sam deploy \
		--s3-bucket s3://$(S3_BUCKET_NAME)/$(STACK_NAME)/ \
		--template-file $(BUILD_DIR)/$(PACKAGED) \
		--stack-name $(STACK_NAME)

.PHONY: destroy
destroy: | clean
	@aws cloudformation delete-stack \
		--stack-name $(STACK_NAME)
