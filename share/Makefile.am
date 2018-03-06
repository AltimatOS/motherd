#  Copyright (C) 2018 Gary Greene
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

install-data-local:
	$(mkinstalldirs) $(DESTDIR)$(pkgdatadir)
	$(FIND) * -type d -print >list.tmp
	for d in `cat list.tmp`; do \
	  $(mkinstalldirs) $(DESTDIR)$(pkgdatadir)/$$d; done
	$(FIND) * \( -name '*.ini' -o -name '*.sql' \) -type f -print >list.tmp
	for f in `cat list.tmp`; do \
	  $(INSTALL_DATA) $$f $(DESTDIR)$(pkgdatadir)/$$f; done
	$(RM) -f list.tmp


uninstall-local:
	$(FIND) * -depth -type d -print >list.tmp
	for d in `cat list.tmp`; do \
	  $(RM) -r $(DESTDIR)$(pkgdatadir)/$$d; \
	done
	$(RM) -f list.tmp


dist-hook:
	$(FIND) * -type d -print >list.tmp
	for d in `cat list.tmp`; do \
	  mkdir $(distdir)/$$d; done
	$(FIND) * \( -name '*.ini' \) -type f -print >list.tmp
	for f in `cat list.tmp`; do \
	  cp $$f $(distdir)/$$f; done
	$(RM) -f list.tmp

