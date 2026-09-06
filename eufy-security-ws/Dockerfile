ARG BUILD_FROM
FROM $BUILD_FROM

ARG EUFY_SECURITY_WS_VERSION

WORKDIR /usr/src/app

# Patched eufy-security-client build (carries PR #959: retry v6 push registration
# after a cached Mega identity is rejected, then persist the refreshed session).
COPY eufy-security-client-build/ /patches/eufy-security-client-build/

RUN \
    set -x \
    # The Alpine 3.23 base image (see build.yaml) ships Node.js 24 in its main
    # repository, so nodejs/npm install directly with no edge-repo workarounds.
    && apk add --no-cache \
        jq \
        nodejs \
        npm \
    && node --version \
    # Fail the build if the base ever regresses below Node 24.
    && case "$(node --version)" in v24.*) ;; *) echo "Expected Node 24, got $(node --version)" >&2; exit 1 ;; esac \
    && npm install --force \
    "eufy-security-ws@${EUFY_SECURITY_WS_VERSION}" \
    \
    # Locate the client the add-on pulled in (robust against hoisting) and swap in
    # our patched build so the retry lands at runtime.
    && CLIENT_DIR="$(node -e "console.log(require.resolve('eufy-security-client/package.json').replace('/package.json',''))")" \
    && echo "Patching eufy-security-client at: ${CLIENT_DIR}" \
    && rm -rf "${CLIENT_DIR}/build" \
    && cp -r /patches/eufy-security-client-build "${CLIENT_DIR}/build"

COPY run.sh /
RUN chmod a+x /run.sh
WORKDIR /data

CMD [ "/run.sh" ]
