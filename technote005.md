# Tech Note 005 - Contexts
### Allow for contexts (pwd, tz, kbd layout, interface language, bin path, display charset, etc.)

* Different texts may have different character set encodings, e.g. utf-8, ISO-Latin1, Shift JIS, Big 5, etc.
* The user may need to switch between keyboard mappigns for input in different languages
* The user may prefer system interaction texts to be displayed in a faimiliar language
* Displayed information may adapt to the appropriate timezone
* Resoures such as binary files, fonts, etc. may depend on the user and on the system
* Should be integrated with Technote 011 Scriptig unifying with environment variables in scripts


