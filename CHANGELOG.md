# Changelog

All notable changes to this project will be documented in this file. See [commit-and-tag-version](https://github.com/absolute-version/commit-and-tag-version) for commit guidelines.

## 1.0.0 (2026-11-23)

### Features

* deploy Prowlarr on Postgres, with a fixed API key, a required login and a claim for /config ([7f11736](https://github.com/tonylea/helm-chart-prowlarr/commit/7f1173615555cc843820cc627e2cf4fa20d2bdf1))

### Bug Fixes

* chown /run and /config from a root init container instead of fsGroup ([3a89425](https://github.com/tonylea/helm-chart-prowlarr/commit/3a894250c4fe34a0140ac794b28604ab70bea118))
