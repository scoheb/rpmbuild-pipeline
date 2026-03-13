ROOT_DIR    := $(shell cd .. && pwd)
SRC_DIR     := $(CURDIR)/src
PATCHES_DIR := $(CURDIR)/patches

# Files and directories to sync from upstream into the repo root.
# .tekton and .gemini are excluded (upstream-only CI/config).
SYNC_ITEMS := \
	CONTRIBUTING.md \
	COPYING \
	README.md \
	diff-flavor.sh \
	docs \
	other-flavors.sh \
	pipeline \
	renovate.json \
	task \
	test-konflux-build-locally

.PHONY: sync check update-upstream regenerate-patches _check-submodule

## sync: Copy upstream sources to repo root and apply fork patches.
sync: _check-submodule
	@echo "==> Removing old synced files..."
	@for item in $(SYNC_ITEMS); do \
		rm -rf "$(ROOT_DIR)/$$item"; \
	done
	@echo "==> Copying upstream files to repo root..."
	@for item in $(SYNC_ITEMS); do \
		if [ -e "$(SRC_DIR)/$$item" ]; then \
			cp -a "$(SRC_DIR)/$$item" "$(ROOT_DIR)/$$item"; \
		else \
			echo "  WARNING: $(SRC_DIR)/$$item does not exist, skipping"; \
		fi; \
	done
	@echo "==> Removing files created by patches..."
	@for patch in $(PATCHES_DIR)/*.patch; do \
		grep -A1 '^--- /dev/null' "$$patch" 2>/dev/null | \
		grep '^+++ b/' | sed 's|^+++ b/||' | while read -r f; do \
			rm -f "$(ROOT_DIR)/$$f"; \
		done; \
	done
	@echo "==> Applying patches..."
	@for patch in $(PATCHES_DIR)/*.patch; do \
		echo "  Applying: $$(basename $$patch)"; \
		git -C "$(ROOT_DIR)" apply "$$patch" || { echo "FAILED: $$patch"; exit 1; }; \
	done
	@echo "==> Sync complete."

## check: Verify that repo root matches upstream/src + patches.
##        Exits non-zero and prints diff on mismatch.
##        Catches both missing patch updates and manual edits.
check: _check-submodule
	$(eval TMPDIR := $(shell mktemp -d))
	@echo "==> Building expected tree in $(TMPDIR)..."
	@for item in $(SYNC_ITEMS); do \
		if [ -e "$(SRC_DIR)/$$item" ]; then \
			cp -a "$(SRC_DIR)/$$item" "$(TMPDIR)/$$item"; \
		fi; \
	done
	@for patch in $(PATCHES_DIR)/*.patch; do \
		echo "  Applying: $$(basename $$patch)"; \
		patch -p1 --no-backup-if-mismatch -d "$(TMPDIR)" < "$$patch" || \
			{ echo "FAILED to apply: $$patch"; rm -rf "$(TMPDIR)"; exit 1; }; \
	done
	@echo "==> Comparing expected tree against repo root..."
	@# Compare every synced item plus any file created by patches (e.g. .gitlab-ci.yml).
	@# We build a combined list from SYNC_ITEMS + any new files the patches created.
	@fail=0; \
	items="$(SYNC_ITEMS)"; \
	for f in "$(TMPDIR)"/.* "$(TMPDIR)"/*; do \
		base=$$(basename "$$f"); \
		case "$$base" in .|..) continue ;; esac; \
		case " $$items " in *" $$base "*) ;; *) items="$$items $$base" ;; esac; \
	done; \
	for item in $$items; do \
		expected="$(TMPDIR)/$$item"; \
		actual="$(ROOT_DIR)/$$item"; \
		if [ -e "$$expected" ] && [ ! -e "$$actual" ]; then \
			echo "MISSING in repo root: $$item"; fail=1; \
		elif [ ! -e "$$expected" ] && [ -e "$$actual" ]; then \
			echo "UNEXPECTED in repo root (not from upstream or patches): $$item"; fail=1; \
		elif [ -e "$$expected" ] && [ -e "$$actual" ]; then \
			diff -ru "$$expected" "$$actual" \
				--label "expected/$$item" --label "actual/$$item" || fail=1; \
		fi; \
	done; \
	rm -rf "$(TMPDIR)"; \
	if [ "$$fail" -ne 0 ]; then \
		echo; \
		echo "FAIL: repo root does not match .hummingbird/src + patches."; \
		echo "If you edited a root file directly, update the corresponding patch instead."; \
		echo "Run 'make -C .hummingbird sync' to regenerate from upstream + patches."; \
		exit 1; \
	fi
	@echo "==> Check passed: repo root is in sync."

## regenerate-patches: Re-export patches so they apply without offsets.
##                     The repo root must already be in sync (run check first).
regenerate-patches: _check-submodule
	$(eval TMPDIR := $(shell mktemp -d))
	@echo "==> Initializing temporary git repo..."
	@git init -q "$(TMPDIR)"
	@git -C "$(TMPDIR)" config user.name "patch-regen"
	@git -C "$(TMPDIR)" config user.email "patch-regen@localhost"
	@for item in $(SYNC_ITEMS); do \
		if [ -e "$(SRC_DIR)/$$item" ]; then \
			cp -a "$(SRC_DIR)/$$item" "$(TMPDIR)/$$item"; \
		fi; \
	done
	@git -C "$(TMPDIR)" add -A && git -C "$(TMPDIR)" commit -q -m "upstream"
	@echo "==> Replaying patches as commits..."
	@count=0; \
	for patch in $(PATCHES_DIR)/*.patch; do \
		echo "  Applying: $$(basename $$patch)"; \
		git -C "$(TMPDIR)" am -q "$$patch" || \
			{ echo "FAILED to apply: $$patch"; rm -rf "$(TMPDIR)"; exit 1; }; \
		count=$$((count + 1)); \
	done; \
	echo "==> Regenerating $$count patch file(s)..."; \
	rm -f $(PATCHES_DIR)/*.patch; \
	git -C "$(TMPDIR)" format-patch \
		--zero-commit --no-signature \
		-o "$(PATCHES_DIR)" \
		"HEAD~$$count..HEAD" > /dev/null
	@rm -rf "$(TMPDIR)"
	@echo "==> Patches regenerated."

## update-upstream: Fetch latest upstream and update the submodule.
##                  After updating, run 'make sync' to regenerate root files
##                  and fix any patches that no longer apply cleanly.
update-upstream:
	@echo "==> Fetching latest upstream..."
	@git -C "$(SRC_DIR)" fetch origin
	@echo "==> Updating submodule to origin/main..."
	@git -C "$(SRC_DIR)" checkout origin/main
	@echo "==> Submodule now at:"
	@git -C "$(SRC_DIR)" log --oneline -1
	@echo
	@echo "Next steps:"
	@echo "  1. make -C .hummingbird sync   (regenerate root files)"
	@echo "  2. Fix any patches that fail to apply"
	@echo "  3. make -C .hummingbird check   (verify everything is consistent)"
	@echo "  4. git add -A && git commit     (commit submodule + root changes)"

_check-submodule:
	@if [ ! -f "$(SRC_DIR)/README.md" ]; then \
		echo "ERROR: .hummingbird/src submodule not initialized."; \
		echo "Run: git submodule update --init"; \
		exit 1; \
	fi
