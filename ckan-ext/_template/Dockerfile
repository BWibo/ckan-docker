###############################################################################
# Build stage
###############################################################################
ARG CKAN_VERSION=2.9.7
ARG BASEIMAGE_VERSION=latest
FROM ghcr.io/keitaroinc/ckan:${CKAN_VERSION} as extbuild
# FROM ckan/ckan:${CKAN_VERSION} as extbuild

ARG CKAN_SPATIAL_VERSION=master
ENV CKAN_SPATIAL_VERSION=${CKAN_SPATIAL_VERSION}

# Switch to the root user
USER root

RUN set -ex && \
#  pip install -U pip && \
  pip install -r https://raw.githubusercontent.com/ckan/ckanext-spatial/${CKAN_SPATIAL_VERSION}/requirements.txt

RUN pip wheel --wheel-dir=/wheels git+https://github.com/ckan/ckanext-spatial.git@${CKAN_SPATIAL_VERSION}#egg=ckanext-spatial

###############################################################################
# Runtime stage
###############################################################################
FROM ghcr.io/tum-gis/ckan-sddi-base:${BASEIMAGE_VERSION} as runtime

# List all extensions here
ENV CKAN__PLUGINS envvars image_view text_view recline_view datastore datapusher spatial_metadata spatial_query

# Switch to the root user
USER root

# Copy python wheels from build stage
COPY --from=extbuild /wheels /srv/app/ext_wheels

# Install and enable the custom extensions, remove wheels
RUN set -ex && \
  pip install --no-index --find-links=/srv/app/ext_wheels ckanext-spatial && \
  ckan config-tool "${APP_DIR}/production.ini" "ckan.plugins = ${CKAN__PLUGINS}" && \
  ckan config-tool "${APP_DIR}/production.ini" "ckanext.spatial.use_postgis_sorting=true" && \
  ckan config-tool "${APP_DIR}/production.ini" "ckanext.spatial.search_backend = solr" && \
  ckan config-tool "${APP_DIR}/production.ini" "ckan.spatial.srid = 4326" && \
  chown -R ckan:ckan /srv/app && \
  rm -rf /srv/app/ext_wheels

# Switch to the ckan user
USER ckan
