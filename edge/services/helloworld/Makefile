# Make targets for building the IBM example helloworld edge service

# This imports the variables from horizon/hzn.json. You can ignore these lines, but do not remove them
-include horizon/.hzn.json.tmp.mk

# Default ARCH to the architecture of this machines (as horizon/golang describes it)
export ARCH ?= $(shell hzn architecture)

# Configurable parameters passed to serviceTest.sh in "test" target
export MATCH:='says: Hello'
export TIME_OUT:=60

# Build the docker image for the current architecture
build:
	docker build --platform linux/$(ARCH) -t $(DOCKER_IMAGE_BASE)_$(ARCH):$(SERVICE_VERSION) -f ./Dockerfile .

# Build the docker image for 3 architectures
build-all-arches:
	ARCH=amd64 $(MAKE) build
	ARCH=arm $(MAKE) build
	ARCH=arm64 $(MAKE) build
	ARCH=ppc64le $(MAKE) build

# Target for travis to test new PRs
test-all-arches:
	ARCH=amd64 $(MAKE) test
	ARCH=arm $(MAKE) test
	ARCH=arm64 $(MAKE) test
	ARCH=ppc64le $(MAKE) test

# Run and verify the service
test: build
	hzn dev service start -S
	@echo 'Testing service...'
	../../../tools/serviceTest.sh $(SERVICE_NAME) $(MATCH) $(TIME_OUT) && \
		{ hzn dev service stop; \
		echo "*** Service test succeeded! ***"; } || \
		{ hzn dev service stop; \
		echo "*** Service test failed! ***"; \
		false ;}

# Publish the service to the Horizon Exchange for the current architecture
publish-service:
	hzn exchange service publish -O -f horizon/service.definition.json

# Target for travis to publish service and pattern after PR is merged
publish:
	ARCH=amd64 $(MAKE) publish-service
	ARCH=amd64 $(MAKE) publish-service-policy
	ARCH=arm $(MAKE) publish-service
	ARCH=arm $(MAKE) publish-service-policy
	ARCH=arm64 $(MAKE) publish-service
	ARCH=arm64 $(MAKE) publish-service-policy
	ARCH=ppc64le $(MAKE) publish-service
	ARCH=ppc64le $(MAKE) publish-service-policy
	$(MAKE) publish-deployment-policy
	hzn exchange pattern publish -f horizon/pattern-all-arches.json

# Build, run and verify, if test succeeds then publish (for the current architecture)
build-test-publish: build test publish-service

# Build/test/publish the service to the Horizon Exchange for 3 architectures and publish a deployment pattern for those architectures
publish-all-arches:
	ARCH=amd64 $(MAKE) build-test-publish
	ARCH=arm $(MAKE) build-test-publish
	ARCH=arm64 $(MAKE) build-test-publish
	ARCH=ppc64le $(MAKE) build-test-publish
	hzn exchange pattern publish -f horizon/pattern-all-arches.json

# target for script - overwrite and pull insitead of push docker image
publish-service-overwrite:
	hzn exchange service publish -O -P --public=true -f horizon/service.definition.json

# Publish Service Policy target for exchange publish script
publish-service-policy:
	hzn exchange service addpolicy -f horizon/service.policy.json $(HZN_ORG_ID)/$(SERVICE_NAME)_$(SERVICE_VERSION)_$(ARCH)

# Publish Deployment Policy target for exchange publish script
publish-deployment-policy:
	hzn exchange deployment addpolicy -f horizon/deployment.policy.json $(HZN_ORG_ID)/policy-$(SERVICE_NAME)_$(SERVICE_VERSION)

# new target for icp exchange to run on startup to publish only
publish-only:
	ARCH=amd64 $(MAKE) publish-service-overwrite
	ARCH=amd64 $(MAKE) publish-service-policy
	ARCH=arm $(MAKE) publish-service-overwrite
	ARCH=arm $(MAKE) publish-service-policy
	ARCH=arm64 $(MAKE) publish-service-overwrite
	ARCH=arm64 $(MAKE) publish-service-policy
	ARCH=ppc64le $(MAKE) publish-service-overwrite
	ARCH=ppc64le $(MAKE) publish-service-policy
	hzn exchange pattern publish -f horizon/pattern-all-arches.json

clean:
	-docker rmi $(DOCKER_IMAGE_BASE)_$(ARCH):$(SERVICE_VERSION) 2> /dev/null || :

clean-all-archs:
	ARCH=amd64 $(MAKE) clean
	ARCH=arm $(MAKE) clean
	ARCH=arm64 $(MAKE) clean
	ARCH=ppc64le $(MAKE) clean

# This imports the variables from horizon/hzn.cfg. You can ignore these lines, but do not remove them.
horizon/.hzn.json.tmp.mk: horizon/hzn.json
	@ hzn util configconv -f $< | sed 's/=/?=/' > $@

.PHONY: build build-all-arches test publish-service build-test-publish publish-all-arches clean clean-all-archs
