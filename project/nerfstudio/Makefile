IMAGE_NAME := nerfstudio:blackwell
WORKSPACE  := /home/ryu/work/3d/project/nerfstudio/folder
CACHE_DIR  := /home/ryu/.cache

build:
	docker build -t $(IMAGE_NAME) .

run:
	docker run --gpus all \
		-v $(WORKSPACE):/workspace/ \
		-v $(CACHE_DIR):/home/user/.cache/ \
		-v $(PWD)/data:/data \
		-p 7007:7007 \
		--rm -it \
		--shm-size=12gb \
		$(IMAGE_NAME)

.PHONY: build run
