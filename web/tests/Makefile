# Environment variables to run tests.

# Database settings can be overwritten by local
# Makefile
PSQL ?= TEST_USE_POSTGRESQL=true
PG8000 ?= CODECHECKER_DB_DRIVER=pg8000
PSYCOPG2 ?= CODECHECKER_DB_DRIVER=psycopg2
DBPORT ?= TEST_DBPORT=5432
DBUNAME ?= TEST_DBUSERNAME=postgres

# Test project configuration, tests are run on these files.
CLANG_VERSION ?= TEST_CLANG_VERSION=stable
TEST_PROJECT ?= TEST_PROJ=$(CURRENT_DIR)/tests/projects

REPO_ROOT ?= REPO_ROOT=$(ROOT)
CC_TEST_WORKSPACE_ROOT ?= $(BUILD_DIR)/workspace

CLEAR_WORKSPACE_CMD = rm -rf $(CC_TEST_WORKSPACE_ROOT)

# Nose test runner configuration options.
NOSECFG = --config .noserc

test: pycodestyle pylint test_unit test_functional

test_in_env: pycodestyle_in_env pylint_in_env test_unit_in_env test_functional_novenv

PYCODESTYLE_TEST_CMD = pycodestyle \
  --exclude=server/codechecker_server/migrations \
  bin codechecker_web tests \
  client/bin client/codechecker_client \
  server/bin server/codechecker_server server/tests \
  tests

pycodestyle:
	$(PYCODESTYLE_TEST_CMD)

pycodestyle_in_env: venv_dev
	$(ACTIVATE_DEV_VENV) && $(PYCODESTYLE_TEST_CMD)

PYLINT_TEST_CMD = pylint ./bin/codechecker-web-version \
  ./codechecker_web \
  ./client/bin/codechecker-cmd ./client/bin/codechecker-store \
  ./client/codechecker_client \
  ./server/bin/codechecker-server ./server/codechecker_server \
  ./server/tests/unit \
	--rcfile=$(ROOT)/.pylintrc \
  --disable=all \
  --enable=logging-format-interpolation,old-style-class

pylint:
	 $(PYLINT_TEST_CMD)

pylint_in_env:
	$(ACTIVATE_DEV_VENV) && $(PYLINT_TEST_CMD)
 
CODECHECKER_CMD = $(BUILD_DIR)/CodeChecker/bin/CodeChecker
SHUTDOWN_SERVER_CMD = echo "Shutting down server..."; \
  HOME="$(CC_TEST_WORKSPACE_ROOT)" ${CODECHECKER_CMD} server -l; \
  HOME="$(CC_TEST_WORKSPACE_ROOT)" ${CODECHECKER_CMD} server \
    --config-directory $(CC_TEST_WORKSPACE_ROOT) \
    --port `cat "$(CC_TEST_WORKSPACE_ROOT)/serverport"` --stop; \
  rm -f "$(CC_TEST_WORKSPACE_ROOT)/serverport"; \
  HOME="$(CC_TEST_WORKSPACE_ROOT)" ${CODECHECKER_CMD} server -l

# Preserve the error status of the previous command but always be able to
# shut down servers.
EXIT_ERROR = { ${SHUTDOWN_SERVER_CMD}; exit 1; }

FUNCTIONAL_TEST_CMD = $(REPO_ROOT) BUILD_DIR=$(BUILD_DIR) $(CLANG_VERSION) $(TEST_PROJECT) \
  nosetests $(NOSECFG) tests/functional \
  && { ${SHUTDOWN_SERVER_CMD}; } || ${EXIT_ERROR}

MAKE_DB_CMD = bash -c 'psql -h localhost \
  -p $${TEST_DBPORT} -U $${TEST_DBUSERNAME} postgres \
  -c "CREATE DATABASE codechecker_config_$${CODECHECKER_DB_DRIVER}"'

DROP_DB_CMD = bash -c 'psql -h localhost \
  -p $${TEST_DBPORT} -U $${TEST_DBUSERNAME} postgres \
  -c "DROP DATABASE IF EXISTS codechecker_config_$${CODECHECKER_DB_DRIVER}"'

RUN_TEST_CMD = $(REPO_ROOT) BUILD_DIR=$(BUILD_DIR) $(CLANG_VERSION) $(TEST_PROJECT) \
  nosetests $(NOSECFG) $(ROOT)/web/${TEST} \
  && { ${SHUTDOWN_SERVER_CMD}; } || ${EXIT_ERROR}

run_test:
	$(RUN_TEST_CMD)

run_test_in_env: venv_dev
	$(ACTIVATE_DEV_VENV) && $(RUN_TEST_CMD)

test_unit: test_unit_server

test_unit_in_env: test_unit_server_in_env

test_functional: test_sqlite test_psql

test_functional_in_env: test_sqlite_in_env test_psql_in_env

test_sqlite:
	python $(ROOT)/scripts/test/check_clang.py || exit 1;
	$(CLEAR_WORKSPACE_CMD) && $(FUNCTIONAL_TEST_CMD)

test_sqlite_in_env: venv_dev
	python $(ROOT)/scripts/test/check_clang.py || exit 1;
	$(ACTIVATE_DEV_VENV) && $(CLEAR_WORKSPACE_CMD) && $(FUNCTIONAL_TEST_CMD)

test_psql: test_psql_psycopg2 test_psql_pg8000

test_psql_in_env: test_psql_psycopg2_in_env test_psql_pg8000_in_env

PSYCOPG2_TEST_CMD = python $(ROOT)/scripts/test/check_clang.py || exit 1; \
  $(DBUNAME) $(DBPORT) $(PSYCOPG2) $(DROP_DB_CMD) && \
  $(DBUNAME) $(DBPORT) $(PSYCOPG2) $(MAKE_DB_CMD) && \
  $(CLEAR_WORKSPACE_CMD) && \
  $(PSQL) $(DBUNAME) $(DBPORT) $(PSYCOPG2) \
  $(FUNCTIONAL_TEST_CMD) && \
  $(DBUNAME) $(DBPORT) $(PSYCOPG2) $(DROP_DB_CMD)

test_psql_psycopg2:
	$(PSYCOPG2_TEST_CMD)

test_psql_psycopg2_in_env: venv_dev
	$(ACTIVATE_DEV_VENV) && $(PSYCOPG2_TEST_CMD)

PG8000_TEST_CMD = python $(ROOT)/scripts/test/check_clang.py || exit 1; \
  $(DBUNAME) $(DBPORT) $(PSYCOPG2) $(DROP_DB_CMD) && \
  $(DBUNAME) $(DBPORT) $(PG8000) $(MAKE_DB_CMD) && \
  $(CLEAR_WORKSPACE_CMD) && \
  $(PSQL) $(DBUNAME) $(DBPORT) $(PG8000) \
  $(FUNCTIONAL_TEST_CMD) && \
  $(DBUNAME) $(DBPORT) $(PG8000) $(DROP_DB_CMD)

test_psql_pg8000:
	$(PG8000_TEST_CMD)

test_psql_pg8000_in_env: venv_dev
	$(ACTIVATE_DEV_VENV) && $(PG8000_TEST_CMD)

test_clean:
	$(CLEAR_WORKSPACE_CMD)
