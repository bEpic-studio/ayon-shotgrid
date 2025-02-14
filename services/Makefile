ADDON_VERSION := $(shell grep "version" ../package.py | cut -d' ' -f3 | tr -d '"')
IMAGE := ynput/ayon-shotgrid-$(SERVICE):$(ADDON_VERSION)
SERVICE = $(error Please specify the service to build with 'SERVICE', for example: 'make build SERVICE=leecher')

default:
	@echo ""
	@echo "AYON Shotgrid $(ADDON_VERSION) Service Builder"
	@echo ""
	@echo "Usage: make SERVICE=[service-name] [target]"
	@echo ""
	@echo "Passing SERVICE is required for any of the targets to work, possible services:"
	@echo ""
	@echo "  leecher - Fetch Shotgrid Events into AYON."
	@echo "  processor - Process 'shotgrid.event's in AYON."
	@echo "  transmitter - Push AYON events to Shotgrid."
	@echo ""
	@echo "Targets:"
	@echo "  build        Build docker image."
	@echo "  build-all    Build docker image for 'leecher', 'procesor' and 'transmitter'."
	@echo "  dist         Push docker image to DockerHub."
	@echo "  dist-all     Push docker image for 'leecher', 'procesor' and 'transmitter'."
	@echo "  clean        Remove local images."
	@echo "  clean-build  Remove local images and build without docker cache."
	@echo "  dev          Run a service locally"


build:
	docker build -t $(IMAGE) -f $(SERVICE)/Dockerfile .

build-all:
	docker build -t ynput/ayon-shotgrid-leecher:$(ADDON_VERSION) -f ./leecher/Dockerfile .
	docker build -t ynput/ayon-shotgrid-processor:$(ADDON_VERSION) -f ./processor/Dockerfile .
	docker build -t ynput/ayon-shotgrid-transmitter:$(ADDON_VERSION) -f ./transmitter/Dockerfile .

dist:
	docker push $(IMAGE)

dist-all:
	docker push ynput/ayon-shotgrid-leecher:$(ADDON_VERSION)
	docker push ynput/ayon-shotgrid-processor:$(ADDON_VERSION)
	docker push ynput/ayon-shotgrid-transmitter:$(ADDON_VERSION)

clean:
	if docker images "$(IMAGE)"; then \
		docker rmi $(IMAGE); \
	fi

clean-build: clean
	docker build --no-cache -t $(IMAGE) -f $(SERVICE)/Dockerfile .

clean-build-all:
	docker build --no-cache -t ynput/ayon-shotgrid-leecher:$(ADDON_VERSION) -f ./leecher/Dockerfile .
	docker build --no-cache -t ynput/ayon-shotgrid-processor:$(ADDON_VERSION) -f ./processor/Dockerfile .
	docker build --no-cache -t ynput/ayon-shotgrid-transmitter:$(ADDON_VERSION) -f ./transmitter/Dockerfile .


dev:
	-ln -s $(CURDIR)/shotgrid_common/* $(CURDIR)/$(SERVICE)
	-docker run --rm -u ayonuser -ti \
		-v $(CURDIR)/shotgrid_common:$(CURDIR)/shotgrid_common:Z \
		-v $(CURDIR)/$(SERVICE):/service:Z \
		--env-file $(CURDIR)/$(SERVICE)/.env \
		--env AYON_ADDON_NAME=shotgrid \
		--env AYON_ADDON_VERSION=$(ADDON_VERSION) \
		--attach=stdin \
		--attach=stdout \
		--attach=stderr \
		$(IMAGE) python -m $(SERVICE)

	find $(CURDIR)/$(SERVICE) -type l -delete

shell:
	docker run --rm -u ayonuser -ti -v $(CURDIR)/$(SERVICE)/$(SERVICE):/service:Z $(IMAGE) /bin/sh
