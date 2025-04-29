CRDOC := $(shell go env GOPATH)/bin/crdoc

$(CRDOC):
	go install fybrik.io/crdoc@latest

OUT_FILE := ../../docs/reference/crds/index.md

KCM_REF ?= main     # or v0.2.0, 0f3e6c1, etc.

TMP_DIR   := $(shell mktemp -d)

api-docs: $(CRDOC)
	@echo "Fetching CRDs for $(KCM_REF)…"
	git clone --depth 1 \
	  --filter=blob:none --sparse \
	  --branch $(KCM_REF) \
	  https://github.com/K0rdent/kcm $(TMP_DIR)/kcm

	git -C $(TMP_DIR)/kcm sparse-checkout set templates/provider/kcm/templates/crds

	$(CRDOC) --resources $(TMP_DIR)/kcm/templates/provider/kcm/templates/crds \
	         --output $(OUT_FILE)
	rm -rf $(TMP_DIR)

