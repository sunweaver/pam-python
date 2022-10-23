PYTHON ?= python3

all:	ctest pam_python.so test-pam_$(PYTHON).pam

WARNINGS=-Wall -Wextra -Wundef -Wshadow -Wpointer-arith -Wbad-function-cast -Wsign-compare -Waggregate-return -Wstrict-prototypes -Wmissing-prototypes -Wmissing-declarations -Werror
#WARNINGS=-Wunreachable-code 	# Gcc 4.1 .. 4.4 are too buggy to make this useful

LIBDIR ?= /lib/security

pam_python.so: pam_python.c setup.py Makefile
	@rm -f "$@"
	@[ ! -e build -o build/lib.*/$@ -nt setup.py -a build/lib.*/$@ -nt Makefile ] || rm -r build
	CFLAGS="$(CFLAGS) $(WARNINGS) -I/usr/local/lib/ " $(PYTHON) setup.py build --debug
	@#CFLAGS="$(CFLAGS) -Og $(WARNINGS)" $(PYTHON) setup.py build --debug
	@#CFLAGS="$(CFLAGS) -Og $(WARNINGS)" Py_DEBUG=1 $(PYTHON) setup.py build --debug
	ln -sf $$(find build/lib.*/pam_python*.so -type f | head -n1) $@

.PHONY: install install-lib
install: install-lib
install-lib:
	mkdir -p $(DESTDIR)$(LIBDIR)
	cp build/lib.*/pam_python.*.so $(DESTDIR)$(LIBDIR)/pam_python.so

.PHONY: clean
clean:
	rm -rf build ctest pam_python.so test-pam_$(PYTHON).pam test.pyc core __pycache__
	[ ! -e /etc/pam.d/test-pam_$(PYTHON).pam ] || { s=$$([ $$(id -u) = 0 ] || echo sudo); $$s rm -f /etc/pam.d/test-pam_$(PYTHON).pam; }
	[ ! -e /etc/pam.d/test-pam_$(PYTHON)-installed.pam ] || { s=$$([ $$(id -u) = 0 ] || echo sudo); $$s rm -f /etc/pam.d/test-pam_$(PYTHON)-installed.pam; }

.PHONY: ctest
ctest:	ctest.c Makefile
	gcc -O0 $(WARNINGS) -g -o $@ ctest.c -lpam

test-pam_$(PYTHON).pam: test-pam_python.pam.in Makefile
	sed "s,\\\$$PWD,$$(pwd),g" "$<" >"$@.tmp"
	mv $@.tmp $@

/etc/pam.d/test-pam_$(PYTHON).pam: test-pam_$(PYTHON).pam
	s=$$([ $$(id -u) = 0 ] || echo sudo); $$s ln -sf $$(pwd)/test-pam_$(PYTHON).pam /etc/pam.d

.PHONY: test
test: pam_python.so ctest /etc/pam.d/test-pam_$(PYTHON).pam
	$(PYTHON) test.py
	./ctest $(PYTHON)

test-pam_$(PYTHON)-installed.pam: test-pam_python.pam.in Makefile
	sed "s,\\\$$PWD/pam_python.so,pam_python.so,;s,\\\$$PWD,$$(pwd),g" "$<" >"$@.tmp"
	mv $@.tmp $@

/etc/pam.d/test-pam_$(PYTHON)-installed.pam: test-pam_$(PYTHON)-installed.pam
	s=$$([ $$(id -u) = 0 ] || echo sudo); $$s ln -sf $$(pwd)/test-pam_$(PYTHON)-installed.pam /etc/pam.d; $$s ln -sf test-pam_$(PYTHON)-installed.pam /etc/pam.d/test-pam_$(PYTHON).pam

.PHONY: installed-test
installed-test: ctest /etc/pam.d/test-pam_$(PYTHON)-installed.pam
	$(PYTHON) test.py
	./ctest $(PYTHON)
