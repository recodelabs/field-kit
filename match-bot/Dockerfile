# Match-bot's web app under gunicorn, for running it on a server.
#
#   docker build -t match-bot match-bot/
#   docker run -p 5000:5000 -v match-bot-data:/data match-bot
#
# The app has no sign-in; put it behind one (e.g. oauth2-proxy) before exposing it. The chat box
# needs the Claude Code CLI, which this image does not include, so it reports the CLI as missing.
FROM python:3.12-slim

# libexpat1: fiona (shapefiles, GeoPackages) needs it.
RUN apt-get update && apt-get install -y --no-install-recommends libexpat1 \
 && rm -rf /var/lib/apt/lists/*

COPY pyproject.toml README.md LICENSE /src/
COPY match_bot /src/match_bot
# The sources are removed after installing, so Flask finds the installed package (and its
# instance folder under /usr/local/var) rather than a copy in the working directory.
RUN pip install --no-cache-dir gunicorn==23.0.0 "/src[all]" && rm -rf /src

# Uploads go to Flask's instance folder, which for an installed package is
# /usr/local/var/match_bot.gui.app-instance; the gazetteer cache to MATCH_BOT_GAZETTEER_DIR.
# Both live under /data, so one volume keeps them.
ENV MATCH_BOT_GAZETTEER_DIR=/data/gazetteer
RUN useradd -r -u 1000 matchbot \
 && mkdir -p /data/gazetteer /data/instance /usr/local/var \
 && ln -s /data/instance /usr/local/var/match_bot.gui.app-instance \
 && chown -R matchbot /data
WORKDIR /data
USER matchbot
EXPOSE 5000

# One worker: the app signs its session cookie with a key made at start-up, so a second worker
# would not recognise the first one's sessions. Threads serve concurrent requests.
CMD ["gunicorn", "--workers", "1", "--threads", "8", "--timeout", "600", "--bind", "0.0.0.0:5000", \
     "match_bot.gui.app:create_app()"]
