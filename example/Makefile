#!/usr/bin/make
# Ultimate Makefile.
# For all your Makefile needs.
# Copyright (C) 2022 Bytez

SHELL = /bin/sh

program_name  = greets
program_id    = com.github.bytezz.greets
program       = greets
package_name  = greets
description   = Greets the user
version_major = 0
version_minor = 0
version_patch = 0

CC ?= gcc

MAKE ?= make

INSTALL ?= install
INSTALL_PROGRAM = $(INSTALL) -D -o $(USER) -g $(USER) -m 755
INSTALL_DATA    = $(INSTALL) -D -o $(USER) -g $(USER) -m 644

srcdir = src

TMPDIR ?= /tmp
prefix = /usr/local
exec_prefix = $(prefix)
bindir = $(exec_prefix)/bin
datarootdir = $(prefix)/share
datadir = $(datarootdir)/$(program_name)
docdir = $(datarootdir)/doc/$(program_name)
sysconfdir = /etc
hicolor = /usr/share/icons/hicolor
applications = /usr/share/applications

all:
	$(CC) -o $(program) $(srcdir)/main.c `pkg-config --cflags --libs gtk+-3.0`

install:
	# Executable $(DESTDIR)$(bindir)
	$(INSTALL_PROGRAM) $(program) $(DESTDIR)$(bindir)/$(program)
	
	# Launcher $(DESTDIR)$(applications)
	$(INSTALL_DATA) data/$(program_id).desktop $(DESTDIR)$(applications)/$(program_id).desktop
	sed -i "s/Name=/Name=$(program_name)/" $(DESTDIR)$(applications)/$(program_id).desktop
	sed -i "s/Comment=/Comment=$(description)/" $(DESTDIR)$(applications)/$(program_id).desktop
	sed -i "s/Exec=/Exec=$(program)/" $(DESTDIR)$(applications)/$(program_id).desktop
	sed -i "s/Icon=/Icon=$(program_name)/" $(DESTDIR)$(applications)/$(program_id).desktop
	
	# Icons $(DESTDIR)$(hicolor)
	$(INSTALL_DATA) data/icons/hicolor/scalable/apps/$(program_name).svg $(DESTDIR)$(hicolor)/scalable/apps/$(program_name).svg
	$(INSTALL_DATA) data/icons/hicolor/symbolic/apps/$(program_name)-symbolic.svg $(DESTDIR)$(hicolor)/symbolic/apps/$(program_name)-symbolic.svg
	
	# Software data $(DESTDIR)$(datadir)
	
	
	# Docs (copyright, changelog, etc.) $(DESTDIR)$(docdir)
	$(INSTALL_DATA) copyright $(DESTDIR)$(docdir)/copyright

uninstall:
	rm -rf $(DESTDIR)$(bindir)/$(program)
	rm -rf $(DESTDIR)$(applications)/$(program_id).desktop
	rm -rf $(DESTDIR)$(hicolor)/scalable/apps/$(program_name).svg
	rm -rf $(DESTDIR)$(hicolor)/symbolic/apps/$(program_name)-symbolic.svg
	rm -rf $(DESTDIR)$(datadir)
	rm -rf $(DESTDIR)$(docdir)

install-user:
	# Install in home dir without root
	$(MAKE) \
		prefix=~/.local \
		sysconfdir=~/.config \
		hicolor=~/.local/share/icons/hicolor \
		applications=~/.local/share/applications \
		install
	sed -i "s/Exec=/Exec=~\/.local\/bin\//" ~/.local/share/applications/$(program_id).desktop

uninstall-user:
	# Uninstall from home dir without root
	$(MAKE) \
		prefix=~/.local \
		sysconfdir=~/.config \
		hicolor=~/.local/share/icons/hicolor \
		applications=~/.local/share/applications \
		uninstall

clean:
	rm -rf $(program)
	rm -rf $(program_name)*.tar.gz
	rm -rf debian
	rm -rf $(program_name)*.deb
	rm -rf rpmbuild
	rm -rf $(program_name).AppDir
	rm -rf *.AppImage

dist:
	rm -rf $(TMPDIR)/$(program_name)-$(version_major).$(version_minor).$(version_patch)
	cp -r . $(TMPDIR)/$(program_name)-$(version_major).$(version_minor).$(version_patch)
	tar -cf $(program_name)-$(version_major).$(version_minor).$(version_patch).tar --directory=$(TMPDIR) $(program_name)-$(version_major).$(version_minor).$(version_patch)
	gzip $(program_name)-$(version_major).$(version_minor).$(version_patch).tar
	rm -rf $(TMPDIR)/$(program_name)-$(version_major).$(version_minor).$(version_patch)

dist-deb: all
	rm -rf debian
	mkdir debian
	$(MAKE) DESTDIR=./debian prefix=/usr install
	mkdir debian/DEBIAN
	cp build-aux/debiancontrol debian/DEBIAN/control
	sed -i "s/Package: /Package: $(package_name)/" debian/DEBIAN/control
	sed -i "s/Version: /Version: $(version_major).$(version_minor).$(version_patch)/" debian/DEBIAN/control
	sed -i "s/Architecture: /Architecture: `uname -m | sed "s/x86_64/amd64/" | sed "s/aarch64/arm64/"`/" debian/DEBIAN/control
	sed -i "s/Description: /Description: $(program_name)\n $(description)/" debian/DEBIAN/control
	dpkg-deb -b debian $(program_name)-$(version_major).$(version_minor).$(version_patch)-`uname -m`.deb

dist-rpm: dist
	# Creating rpm build dir
	rm -rf rpmbuild
	mkdir rpmbuild
	mkdir rpmbuild/BUILD
	mkdir rpmbuild/BUILDROOT
	mkdir rpmbuild/RPMS
	mkdir rpmbuild/SOURCES
	mkdir rpmbuild/SPECS
	mkdir rpmbuild/SRPMS
	# Importing source code inside sources
	mv $(program_name)-$(version_major).$(version_minor).$(version_patch).tar.gz rpmbuild/SOURCES
	# Updating release number
	awk 'BEGIN{ FS=OFS="  " } /Release:    /{ match($$NF, /^[0-9]+/); $$NF=($$NF+1) substr($$NF, RLENGTH+1) }1' build-aux/rpmspec > $(TMPDIR)/rpmspec && cat $(TMPDIR)/rpmspec > build-aux/rpmspec
	# Copying rpm spec inside rpm build dir
	cp build-aux/rpmspec rpmbuild/SPECS/$(program_name).spec
	# Setting software version, build arch and source file name inside rpm spec
	sed -i "s/Name:       /Name:       $(program_name)/" rpmbuild/SPECS/$(program_name).spec
	sed -i "s/Version:    /Version:    $(version_major).$(version_minor).$(version_patch)/" rpmbuild/SPECS/$(program_name).spec
	sed -i "s/Summary:    /Summary:    $(description)/" rpmbuild/SPECS/$(program_name).spec
	sed -i "s/Source0:    /Source0:    $(program_name)-$(version_major).$(version_minor).$(version_patch).tar.gz/" rpmbuild/SPECS/$(program_name).spec
	sed -i "s/BuildArch:  /BuildArch:  `uname -m`/" rpmbuild/SPECS/$(program_name).spec
	# Build rpm
	rpmbuild --define "_topdir `pwd`/rpmbuild" -ba rpmbuild/SPECS/$(program_name).spec

dist-app: all
	rm -rf $(program_name).AppDir
	mkdir $(program_name).AppDir
	$(MAKE) DESTDIR=./$(program_name).AppDir install
	# Setting AppImage icon
	$(INSTALL_DATA) data/icons/$(program_name).svg $(program_name).AppDir/$(program_name).svg
	# Setting AppImage launcher
	$(INSTALL_DATA) data/$(program_id).desktop $(program_name).AppDir/$(program_name).desktop
	sed -i "s/Name=/Name=$(program_name)/" $(program_name).AppDir/$(program_name).desktop
	sed -i "s/Comment=/Comment=$(description)/" $(program_name).AppDir/$(program_name).desktop
	sed -i "s/Exec=/Exec=$(program)/" $(program_name).AppDir/$(program_name).desktop
	sed -i "s/Icon=/Icon=$(program_name)/" $(program_name).AppDir/$(program_name).desktop
	# ↓ Dependencies ↓
	$(INSTALL_DATA) /usr/lib/x86_64-linux-gnu/libgtk-3.so $(program_name).AppDir/usr/lib/libgtk-3.so
	# ↑ End dependencies ↑
	echo "#!/bin/bash\ncdir=\$$(dirname \$$BASH_SOURCE)\n\$$cdir$(bindir)/$(program)" > $(program_name).AppDir/AppRun
	chmod +x $(program_name).AppDir/AppRun
	ARCH=`uname -m` appimagetool $(program_name).AppDir

.PHONY: all install uninstall clean
