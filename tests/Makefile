# NOTE: `cabal test` will take care to build the local `alex`
# executable and place it into $PATH for us to pick up.
#
# If it doesn't look like the alex binary in $PATH comes from the
# build tree, then we'll fall back to pointing to
# ../dist/build/alex/alex to support running tests via "runghc
# Setup.hs test".
#
# If ALEX has been set outside, e.g. in the environment, we trust this setting.
# This way, we can pass in the correct Alex executable from a CI environment
# without danger of it being "fixed" by the logic below.
# [2021-06-15, PR #189](https://github.com/simonmar/alex/pull/189)
#
ifndef ALEX
ALEX=$(shell which alex)
ifeq "$(filter $(dir $(shell pwd))%,$(ALEX))" ""
ALEX=../dist/build/alex/alex
endif
endif

# NOTE: This assumes that a working `ghc` is on $PATH; this may not
# necessarily be the same GHC used by `cabal` for building `alex`.
#
# Again, if HC has been set in the environment (e.g. by the CI), we keep this setting.
# [2021-06-15, PR #189](https://github.com/simonmar/alex/pull/189)
#
HC ?= ghc

# Some GHC warnings are only available from a certain version on
# Get the GHC version
GHC_VERSION:=$(shell $(HC) --numeric-version)
GHC_VERSION_WORDS=$(subst ., ,$(GHC_VERSION))
GHC_MAJOR_VERSION=$(word 1,$(GHC_VERSION_WORDS))
GHC_MINOR_VERSION=$(word 2,$(GHC_VERSION_WORDS))
# -fwarn-incomplete-uni-patterns only from 7.4
WARNS_FOR_GHC_GTEQ_7_4=-fwarn-incomplete-uni-patterns
WARNS_FOR_GHC_LT_7_4=-fno-warn-lazy-unlifted-bindings
WARNS_DEP_GHC_GTEQ_7_4:=$(shell if [ $(GHC_MAJOR_VERSION) -gt 7 -o $(GHC_MAJOR_VERSION) -ge 7 -a $(GHC_MINOR_VERSION) -ge 4 ]; then echo "$(WARNS_FOR_GHC_GTEQ_7_4)"; else echo "$(WARNS_FOR_GHC_LT_7_4)"; fi)

HC_OPTS=-Wall $(WARNS_DEP_GHC_GTEQ_7_4) -fno-warn-missing-signatures -fno-warn-unused-imports -fno-warn-tabs -Werror

.PRECIOUS: %.n.hs %.g.hs %.o %.exe %.bin

ifeq "$(TARGETPLATFORM)" "i386-unknown-mingw32"
HS_PROG_EXT = .exe
else
HS_PROG_EXT = .bin
endif

TESTS = \
        basic_typeclass.x \
        basic_typeclass_bytestring.x \
        default_typeclass.x \
        gscan_typeclass.x \
        issue_71.x \
        issue_119.x \
        issue_141.x \
        issue_197.x \
        monad_typeclass.x \
        monad_typeclass_bytestring.x \
        monadUserState_typeclass.x \
        monadUserState_typeclass_bytestring.x \
        null.x \
        posn_typeclass.x \
        posn_typeclass_bytestring.x \
        strict_typeclass.x \
        simple.x \
        tokens.x \
        tokens_bytestring.x \
        tokens_bytestring_unicode.x \
        tokens_gscan.x \
        tokens_monad_bytestring.x \
        tokens_monadUserState_bytestring.x \
        tokens_posn.x \
        tokens_posn_bytestring.x \
        tokens_scan_user.x \
        tokens_strict_bytestring.x \
        unicode.x

# NOTE: `cabal` will set the `alex_datadir` env-var accordingly before invoking the test-suite
#TEST_ALEX_OPTS = --template=../data/
TEST_ALEX_OPTS=

%.n.hs : %.x
	$(ALEX) $(TEST_ALEX_OPTS) $< -o $@

%.g.hs : %.x
	$(ALEX) $(TEST_ALEX_OPTS) -g $< -o $@

CLEAN_FILES += *.n.hs *.g.hs *.info *.hi *.o *.bin *.exe

ALL_TEST_HS = $(shell echo $(TESTS) | sed -e 's/\([^\. ]*\)\.\(l\)\{0,1\}x/\1.n.hs \1.g.hs/g')

ALL_TESTS = $(patsubst %.hs, %.run, $(ALL_TEST_HS))

%.run : %$(HS_PROG_EXT)
	./$<

%$(HS_PROG_EXT) : %.hs
	$(HC) $(HC_OPTS) -package array -package bytestring $($*_LD_OPTS) $< -o $@

all :: $(ALL_TESTS)

.PHONY: clean debug

clean:
	rm -f $(CLEAN_FILES)

# NOTE: The `../dist` path belows don't aren't accurate anymore for recent cabals
interact:
	ghci -cpp -i../src -i../dist/build/autogen -i../dist/build/alex/alex-tmp Main -fbreak-on-exception
# -args='--template=.. simple.x -o simple.n.hs'
# :set args --template=.. simple.x -o simple.n.hs

debug :
	@echo HC_OPTS=$(HC_OPTS)
