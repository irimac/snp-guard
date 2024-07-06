IMAGE ?= snp-builder
DOCKERFILE ?= snp-builder.dockerfile
NAME ?= snp-builder
OUTPUT_PATH ?= ../build

STATUS = $(shell docker inspect $(NAME) | grep Status | cut -d':' -f2 | sed 's/,//g' | xargs)
EXIT_CODE = $(shell docker inspect $(NAME) | grep ExitCode | cut -d':' -f2 | sed 's/,//g' | xargs)

USE_STABLE_SNAPSHOTS ?= 0

# Check if http_proxy is defined and set the build-arg accordingly
ifdef http_proxy
BUILD_ARGS := --build-arg http_proxy=$(http_proxy) --build-arg https_proxy=$(http_proxy)
RUN_ARGS := -e http_proxy=$(http_proxy) -e https_proxy=$(http_proxy)
else
BUILD_ARGS := 
RUND_ARGS :=
endif

image:
	docker build $(BUILD_ARGS) -t $(IMAGE) -f $(DOCKERFILE) .

build: clean
	docker run -v /boot:/boot -d --name $(NAME) -e USE_STABLE_SNAPSHOTS=$(USE_STABLE_SNAPSHOTS) $(RUN_ARGS) $(IMAGE)

get_files: check_status $(OUTPUT_PATH)
	docker cp $(NAME):/usr/src/app/snp-release.tar.gz $(OUTPUT_PATH)

$(OUTPUT_PATH):
	mkdir -p $(OUTPUT_PATH)

check_status:
	@echo $(STATUS) | grep -q exited && echo $(EXIT_CODE) | grep -q 0 || (echo "Error! Status: $(STATUS) ExitCode: $(EXIT_CODE)" && exit 1)

clean:
	@docker stop $(IMAGE) ||  true
	@docker rm $(IMAGE) || true