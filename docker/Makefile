

all: ensure-build container-rm
	$(MAKE) run ARG="$(ARG)"

# ---------------------------------------------------------------------------- #
#                            Docker Builtin Commands                           #
# ---------------------------------------------------------------------------- #

# Build the image
build:
	docker build -t $(IMAGE_NAME):latest .

# Save the image to a tar file
save:
	mkdir -p tmp/image
	docker save $(IMAGE_NAME):latest -o tmp/image/$(IMAGE_NAME).tar
	@echo "Image saved to tmp/image/$(IMAGE_NAME).tar"

# Load the image from a tar file
load:
	docker load -i tmp/image/$(IMAGE_NAME).tar

# Run the container
# Note: The following command requires X11 forwarding to be set up on your host machine.
run:
	@if [ -z "$$(docker images -q $(IMAGE_NAME):latest)" ]; then \
		echo "Image $(IMAGE_NAME):latest not found. Please build the image first."; \
		exit 1; \
	fi
	mkdir -p tmp/.gradle
	xhost +local:root
	-docker run -it \
		--privileged \
		--net=host \
		--env="DISPLAY" \
		-e XDG_RUNTIME_DIR=/tmp \
		-e QT_X11_NO_MITSHM=1 \
		-v /dev/bus/usb:/dev/bus/usb \
		-v /tmp/.X11-unix:/tmp/.X11-unix:rw \
		--device=/dev/dri \
		--group-add video \
		--mount type=bind,source=$(CURDIR)/../assets,target=/root/assets \
		--mount type=bind,source=$(CURDIR)/../examples,target=/root/examples \
		--mount type=bind,source=$(CURDIR)/../scripts,target=/root/scripts \
		--mount type=bind,source=$(CURDIR)/../src,target=/root/src \
		--name $(IMAGE_NAME) \
		$(IMAGE_NAME):latest $(ARG)
	xhost -local:root

# Start the container
start:
	-docker start $(IMAGE_NAME)
	-docker exec -it $(IMAGE_NAME) /bin/zsh

# Stop the container
stop:
	-docker stop $(IMAGE_NAME)

container-rm: stop
	-docker container rm $(IMAGE_NAME)

rmi:
	-docker rmi $(IMAGE_NAME):latest

# Attach to the running container
attach:
	-docker exec -it $(IMAGE_NAME) /bin/zsh

# ---------------------------------------------------------------------------- #
#                                 Tool Commands                                #
# ---------------------------------------------------------------------------- #

# Remove the container and image
clean: stop container-rm rmi
	echo "Cleaned up the container and image."

### Build
# Build the image if network is present
#     If the cached image is different from the current image, replace the cached image
#     If the cached image is the same as the current image, do nothing
# If not, find the cached image
# If cached image is not present, check if the image is already present
# If all fail, exit with error code
ensure-build:
	@set -e; \
	if ping -c 1 -W 1 8.8.8.8 > /dev/null 2>&1; then \
		echo -e "\e[1;36mNetwork is up, building the image\e[0m"; \
		$(MAKE) build; \
		if [ -f "tmp/dockerfile-hash.txt" ] && sha256sum -c tmp/dockerfile-hash.txt > /dev/null 2>&1; then \
			echo -e "\e[1;36mCached image is up to date\e[0m"; \
		else \
			echo -e "\e[1;36mCached image is different, replacing the cached image\e[0m"; \
			$(MAKE) save; \
			sha256sum Dockerfile > tmp/dockerfile-hash.txt; \
		fi; \
	else \
		echo -e "\e[1;36mNetwork is down, finding the cached image\e[0m"; \
		if [ -f "tmp/image/$(IMAGE_NAME).tar" ]; then \
			echo -e "\e[1;36mCached image found, loading the image\e[0m"; \
			$(MAKE) load; \
		else \
			echo -e "\e[1;36mCached image not found, checking if the image is already present\e[0m"; \
			if [ -z "$$(docker images -q $(IMAGE_NAME):latest)" ]; then \
				echo -e "\e[31mImage not found, please build the image when connected to a network\e[0m"; \
				exit 1; \
			else \
				echo -e "\e[1;36mImage found in docker.\e[0m"; \
			fi; \
		fi; \
	fi
