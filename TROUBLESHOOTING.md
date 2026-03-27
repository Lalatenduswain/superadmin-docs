# Troubleshooting Guide

> Solutions for common issues encountered when running the SuperAdmin SaaS platform.

Each issue is documented with four sections: **Symptom** (what you observe), **Cause** (why it happens), **Solution** (how to fix it), and **Prevention** (how to avoid it in the future).

---

## Table of Contents

1. [Database Connection Issues](#1-database-connection-issues)
2. [Redis/Valkey Connection Failures](#2-redisvalkey-connection-failures)
3. [JWT Token Errors](#3-jwt-token-errors)
4. [CORS Issues](#4-cors-issues)
5. [Rate Limiting False Positives](#5-rate-limiting-false-positives)
6. [File Upload Failures](#6-file-upload-failures)
7. [MinIO Connectivity](#7-minio-connectivity)
8. [Docker Container Issues](#8-docker-container-issues)
9. [Kubernetes Pod Crashes](#9-kubernetes-pod-crashes)
10. [SSL Certificate Problems](#10-ssl-certificate-problems)
11. [Memory Leaks](#11-memory-leaks)
12. [Performance Degradation](#12-performance-degradation)

---

## 1. Database Connection Issues

### 1.1 Connection Refused

**Symptom:**
```
Error: connect ECONNREFUSED 127.0.0.1:5432
```
The application fails to start or throws errors on every request.

**Cause:** PostgreSQL is not running, not listening on the expected port, or the host address is incorrect.

**Solution:**
```bash
# Check if PostgreSQL is running
sudo systemctl status postgresql
# or for Docker
docker ps | grep postgres

# Verify it is listening on the correct port
sudo ss -tlnp | grep 5432

# If not running, start it
sudo systemctl start postgresql
# or
docker start superadmin-postgres

# If running but not listening on 5432, check postgresql.conf
sudo grep "^port" /etc/postgresql/18/main/postgresql.conf

# Verify .env matches the actual host
# If PostgreSQL runs in Docker and the app runs on the host:
#   DB_HOST=localhost (or 127.0.0.1)
# If both run in Docker Compose:
#   DB_HOST=postgres (the service name)
```

**Prevention:** Use Docker Compose health checks so the application waits for PostgreSQL to be ready before starting. Add a startup connection retry loop with exponential backoff.

---

### 1.2 Authentication Failed

**Symptom:**
```
error: password authentication failed for user "superadmin"
```

**Cause:** The database password in `.env` does not match the password configured in PostgreSQL, or the `pg_hba.conf` file does not allow the connection method.

**Solution:**
```bash
# Reset the password in PostgreSQL
sudo -u postgres psql -c "ALTER ROLE superadmin WITH PASSWORD 'new_password';"

# Update .env to match
# DB_PASSWORD=new_password

# Check pg_hba.conf for the authentication method
sudo cat /etc/postgresql/18/main/pg_hba.conf | grep superadmin
# Ensure the line uses scram-sha-256 (not md5 or reject)

# Reload PostgreSQL config
sudo systemctl reload postgresql
```

**Prevention:** Store database credentials in a secrets manager (Docker secrets or Kubernetes secrets) and reference them consistently. Avoid manual password changes without updating all references.

---

### 1.3 Connection Pool Exhaustion

**Symptom:**
```
Error: remaining connection slots are reserved for non-replication superuser connections
```
Or the application becomes unresponsive with requests timing out while database CPU is low.

**Cause:** Too many connections are held open. This can happen when the pool size across all application replicas exceeds PostgreSQL `max_connections`, or when connections leak due to unhandled errors.

**Solution:**
```bash
# Check current connections
sudo -u postgres psql -c "SELECT count(*) FROM pg_stat_activity;"

# Check connections by application
sudo -u postgres psql -c "SELECT usename, client_addr, state, count(*) FROM pg_stat_activity GROUP BY usename, client_addr, state ORDER BY count DESC;"

# Kill idle connections older than 10 minutes
sudo -u postgres psql -c "SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE state = 'idle' AND query_start < now() - interval '10 minutes' AND usename = 'superadmin';"

# Increase max_connections if needed (requires restart)
# In postgresql.conf: max_connections = 300
sudo systemctl restart postgresql
```

**Prevention:** Set `DB_POOL_MAX` so that `total_pods * DB_POOL_MAX < max_connections - 10`. Use connection pool monitoring via the `/api/v1/monitoring/database-health` endpoint. Configure `idle_in_transaction_session_timeout` in PostgreSQL to kill abandoned transactions.

---

### 1.4 Slow Queries

**Symptom:** API response times increase gradually. The `/api/v1/monitoring/database-health` endpoint reports slow queries.

**Cause:** Missing indexes, table bloat after many updates/deletes, or queries scanning large tables without proper filtering.

**Solution:**
```bash
# Identify slow queries
sudo -u postgres psql -d superadmin_db -c "
SELECT query, calls, mean_exec_time, total_exec_time
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;"

# Check for missing indexes
sudo -u postgres psql -d superadmin_db -c "
SELECT relname, seq_scan, seq_tup_read, idx_scan
FROM pg_stat_user_tables
WHERE seq_scan > 1000 AND idx_scan < seq_scan / 10
ORDER BY seq_tup_read DESC;"

# Analyze table statistics
sudo -u postgres psql -d superadmin_db -c "ANALYZE;"

# Rebuild bloated indexes
sudo -u postgres psql -d superadmin_db -c "REINDEX DATABASE superadmin_db;"
```

**Prevention:** Run `ANALYZE` weekly via a scheduled job. Monitor query performance through Grafana dashboards. Add indexes for any column used in `WHERE`, `JOIN`, or `ORDER BY` clauses. Use `EXPLAIN ANALYZE` during development to verify query plans.

---

## 2. Redis/Valkey Connection Failures

### 2.1 Connection Refused

**Symptom:**
```
Error: connect ECONNREFUSED 127.0.0.1:6379
```

**Cause:** Redis/Valkey is not running or not bound to the expected interface.

**Solution:**
```bash
# Check if Redis is running
sudo systemctl status redis-server
# or
docker ps | grep redis

# Start if not running
sudo systemctl start redis-server
# or
docker start superadmin-redis

# Check binding
redis-cli -h 127.0.0.1 -p 6379 ping

# If Redis is bound only to 127.0.0.1 but the app is in Docker,
# edit redis.conf to bind to 0.0.0.0 (within Docker network only)
```

**Prevention:** Use Docker Compose health checks. Configure the application to retry Redis connections on startup.

---

### 2.2 Authentication Required

**Symptom:**
```
NOAUTH Authentication required
```
or
```
ERR invalid password
```

**Cause:** Redis is configured with `requirepass` but the application is not sending a password, or the passwords do not match.

**Solution:**
```bash
# Test the password
redis-cli -a your_password ping
# Should return: PONG

# If the password is unknown, check the Redis config
sudo grep "requirepass" /etc/redis/redis.conf

# Update .env with the correct password
# REDIS_PASSWORD=correct_password

# Restart the application
npm run dev
```

**Prevention:** Define Redis credentials in a shared secrets file. Run password verification as part of the health check.

---

### 2.3 Memory Exceeded

**Symptom:**
```
OOM command not allowed when used memory > 'maxmemory'
```
Write operations to Redis fail while reads continue to work.

**Cause:** Redis has reached its configured `maxmemory` limit. If the eviction policy is `noeviction`, no keys are removed and writes are rejected.

**Solution:**
```bash
# Check current memory usage
redis-cli -a $REDIS_PASSWORD INFO memory

# Check maxmemory setting
redis-cli -a $REDIS_PASSWORD CONFIG GET maxmemory

# Increase maxmemory
redis-cli -a $REDIS_PASSWORD CONFIG SET maxmemory 2gb

# Or switch to an eviction policy that removes old keys
redis-cli -a $REDIS_PASSWORD CONFIG SET maxmemory-policy allkeys-lru

# Clear expired sessions and stale cache to free memory immediately
redis-cli -a $REDIS_PASSWORD --scan --pattern "sa:cache:*" | xargs redis-cli -a $REDIS_PASSWORD DEL
```

**Prevention:** Set `maxmemory-policy` to `allkeys-lru` in production so Redis evicts least-recently-used keys when memory is full. Monitor Redis memory usage in Grafana and set alerts at 80% utilization.

---

## 3. JWT Token Errors

### 3.1 Token Expired

**Symptom:**
```json
{
  "success": false,
  "error": { "code": "AUTH_TOKEN_EXPIRED", "message": "Access token has expired" }
}
```

**Cause:** The access token's `exp` claim is in the past. Access tokens have a short lifetime (default: 15 minutes) by design.

**Solution:**

Client-side: implement automatic token refresh. When receiving a 401 with `AUTH_TOKEN_EXPIRED`, call the `/api/v1/auth/refresh` endpoint with the refresh token to obtain a new access token, then retry the original request.

```typescript
// Client-side refresh logic
async function fetchWithRefresh(url: string, options: RequestInit): Promise<Response> {
  let response = await fetch(url, options);

  if (response.status === 401) {
    const body = await response.json();
    if (body.error?.code === 'AUTH_TOKEN_EXPIRED') {
      const refreshResponse = await fetch('/api/v1/auth/refresh', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ refreshToken: getStoredRefreshToken() }),
      });

      if (refreshResponse.ok) {
        const { data } = await refreshResponse.json();
        storeAccessToken(data.accessToken);
        options.headers = {
          ...options.headers,
          Authorization: `Bearer ${data.accessToken}`,
        };
        response = await fetch(url, options);
      }
    }
  }

  return response;
}
```

**Prevention:** Set `JWT_ACCESS_EXPIRY` to a reasonable value (15-30 minutes). Implement proactive token refresh (refresh when the token is within 2 minutes of expiry, rather than waiting for a 401).

---

### 3.2 Invalid Signature

**Symptom:**
```json
{
  "success": false,
  "error": { "code": "AUTH_TOKEN_INVALID", "message": "Invalid token signature" }
}
```

**Cause:** The `JWT_SECRET` on the server that issued the token differs from the `JWT_SECRET` on the server validating it. This commonly occurs after rotating the secret, during deployments with mismatched environment variables, or when using a token from a different environment (e.g., staging token on production).

**Solution:**
```bash
# Verify all pods/instances use the same JWT_SECRET
# In Kubernetes:
kubectl get secret superadmin-jwt -n superadmin -o jsonpath='{.data.secret}' | base64 -d

# In Docker Compose:
cat secrets/jwt_secret.txt

# If the secret was rotated, all existing tokens are invalid.
# Force-logout all users to clear old tokens:
curl -X POST http://localhost:3000/api/v1/auth/bulk-force-logout \
  -H "Authorization: Bearer $SUPER_ADMIN_TOKEN"
```

**Prevention:** Store `JWT_SECRET` in a central secrets manager. When rotating secrets, implement a grace period where both old and new secrets are accepted, then phase out the old secret after all tokens have expired.

---

### 3.3 Token Blacklisted

**Symptom:**
```json
{
  "success": false,
  "error": { "code": "AUTH_TOKEN_REVOKED", "message": "Token has been revoked" }
}
```

**Cause:** The token was explicitly revoked via logout or force-logout. The token ID is stored in the Redis blacklist.

**Solution:** Obtain a new token by logging in again. This is expected behavior after logout or security-triggered session revocation.

**Prevention:** Ensure the client clears stored tokens on logout and redirects to the login page when receiving this error.

---

## 4. CORS Issues

### 4.1 Origin Not Allowed

**Symptom:**
```
Access to XMLHttpRequest at 'https://api.example.com/api/v1/users' from origin
'https://app.example.com' has been blocked by CORS policy: No 'Access-Control-Allow-Origin'
header is present on the response.
```

**Cause:** The requesting origin is not included in the `CORS_ORIGINS` environment variable.

**Solution:**
```bash
# Add the origin to CORS_ORIGINS (comma-separated list)
# In .env:
CORS_ORIGINS=https://app.example.com,https://admin.example.com

# Restart the application
npm run dev
```

**Prevention:** Maintain a complete list of all frontend origins in `CORS_ORIGINS`. In development, include `http://localhost:5173` and `http://localhost:3000`. In production, list only the exact production domains.

---

### 4.2 Preflight Request Fails

**Symptom:**
```
CORS preflight request did not succeed. Status code: 405 or 403
```
The browser sends an `OPTIONS` request that is rejected.

**Cause:** The server does not handle `OPTIONS` requests, or a middleware (rate limiter, auth) is blocking the preflight before the CORS middleware runs.

**Solution:** Ensure the CORS middleware is registered before authentication and rate limiting middleware. The middleware execution order should be:

1. Raw body parser
2. CORS middleware (handles OPTIONS preflight)
3. Security headers
4. Rate limiter
5. Authentication
6. Route handlers

Check that the CORS middleware allows the required headers:

```env
# These headers must be allowed:
# Authorization, Content-Type, X-Requested-With, X-CSRF-Token
```

**Prevention:** Follow the middleware execution order documented in Section 42 of `SUPER_ADMIN_DOCUMENTATION.md`. Test CORS configuration with `curl`:

```bash
curl -v -X OPTIONS https://api.example.com/api/v1/users \
  -H "Origin: https://app.example.com" \
  -H "Access-Control-Request-Method: GET" \
  -H "Access-Control-Request-Headers: Authorization"
```

---

### 4.3 Credentials Not Included

**Symptom:** Authenticated requests work in Postman but fail in the browser. The `Authorization` header is not sent.

**Cause:** The client is not configured to include credentials, or the server does not set `Access-Control-Allow-Credentials: true`.

**Solution:**

Server side: ensure `CORS_CREDENTIALS=true` in `.env`.

Client side:
```typescript
// Fetch API
fetch(url, { credentials: 'include', headers: { Authorization: `Bearer ${token}` } });

// Axios
axios.defaults.withCredentials = true;
```

**Prevention:** Set `CORS_CREDENTIALS=true` by default and document it in the client integration guide.

---

## 5. Rate Limiting False Positives

### 5.1 Legitimate Users Rate Limited

**Symptom:** Users receive `429 Too Many Requests` during normal usage. The `X-RateLimit-Remaining` header shows `0`.

**Cause:** Multiple users share the same IP address (corporate NAT, VPN, or proxy). The rate limiter counts all requests from that IP as a single source.

**Solution:**
```bash
# Clear the rate limit for the affected IP
curl -X DELETE http://localhost:3000/api/v1/security/rate-limits/ip/203.0.113.50 \
  -H "Authorization: Bearer $ADMIN_TOKEN"

# Add the corporate IP to the rate limit allowlist with a higher threshold
# This is configured in system settings
curl -X PUT http://localhost:3000/api/v1/config/settings \
  -H "Authorization: Bearer $ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"rateLimiting": {"trustedProxies": ["203.0.113.0/24"], "trustedProxyMultiplier": 10}}'
```

**Prevention:** Configure rate limiting to use the `X-Forwarded-For` header when behind a trusted proxy. Set `trustedProxies` in the system configuration to identify reverse proxies. For known corporate networks, increase the rate limit multiplier.

---

### 5.2 Rate Limits Not Resetting

**Symptom:** A user remains rate limited even after waiting beyond the reset window.

**Cause:** The Redis key TTL is not set correctly, or the application clock is skewed from the Redis server clock.

**Solution:**
```bash
# Check the TTL of the rate limit key
redis-cli -a $REDIS_PASSWORD TTL "sa:rate:192.168.1.100:/api/v1/users"

# If TTL is -1 (no expiry), the key was set without a TTL
redis-cli -a $REDIS_PASSWORD EXPIRE "sa:rate:192.168.1.100:/api/v1/users" 900

# Or clear the key entirely
redis-cli -a $REDIS_PASSWORD DEL "sa:rate:192.168.1.100:/api/v1/users"

# Check for clock skew
date && redis-cli -a $REDIS_PASSWORD TIME
```

**Prevention:** Ensure rate limit keys are always set with an explicit TTL. Use `SET key value EX seconds` rather than `SET` followed by `EXPIRE`. Monitor Redis key counts to detect TTL issues.

---

## 6. File Upload Failures

### 6.1 File Too Large

**Symptom:**
```json
{
  "success": false,
  "error": { "code": "FILE_TOO_LARGE", "message": "File exceeds maximum size of 50 MB" }
}
```

**Cause:** The uploaded file exceeds the configured maximum file size.

**Solution:**

If the file should be allowed, increase the limit:

```bash
curl -X PUT http://localhost:3000/api/v1/config/settings \
  -H "Authorization: Bearer $ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"storage": {"maxFileSize": 104857600}}'
```

Also verify that the reverse proxy allows the upload size:

```yaml
# Traefik
http:
  middlewares:
    upload-size:
      buffering:
        maxRequestBodyBytes: 104857600

# Nginx (if used)
# client_max_body_size 100m;
```

**Prevention:** Set the file size limit to match your actual requirements. Configure the reverse proxy limit to match or exceed the application limit.

---

### 6.2 MIME Type Not Allowed

**Symptom:**
```json
{
  "success": false,
  "error": { "code": "FILE_TYPE_NOT_ALLOWED", "message": "File type application/x-msdownload is not allowed" }
}
```

**Cause:** The file's detected MIME type is not in the allowlist. The application checks the file's magic bytes (not just the extension) to determine the true MIME type.

**Solution:**

If the MIME type should be allowed, add it to the configuration:

```bash
curl -X PUT http://localhost:3000/api/v1/config/settings \
  -H "Authorization: Bearer $ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"storage": {"allowedMimeTypes": ["image/png", "image/jpeg", "application/pdf", "text/csv", "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"]}}'
```

**Prevention:** Document the list of allowed MIME types for users. Review the allowlist periodically to ensure it covers all legitimate use cases while excluding executable file types.

---

### 6.3 Virus Detected

**Symptom:**
```json
{
  "success": false,
  "error": { "code": "FILE_VIRUS_DETECTED", "message": "Malware detected in uploaded file" }
}
```

**Cause:** ClamAV detected malware signatures in the uploaded file.

**Solution:** This is expected behavior. Do not bypass virus scanning. Inform the user that the file was rejected for security reasons and advise them to scan the file locally.

If you suspect a false positive:

```bash
# Scan the file manually with ClamAV
clamscan --verbose /path/to/file

# Update virus definitions
sudo freshclam

# Rescan after updating
clamscan --verbose /path/to/file
```

**Prevention:** Keep ClamAV virus definitions updated daily. Configure `freshclam` as a cron job.

---

## 7. MinIO Connectivity

### 7.1 Bucket Not Found

**Symptom:**
```json
{
  "success": false,
  "error": { "code": "STORAGE_ERROR", "message": "The specified bucket does not exist" }
}
```

**Cause:** The required bucket has not been created in MinIO, or the bucket name in the configuration does not match.

**Solution:**
```bash
# List existing buckets
mc alias set local http://localhost:9000 $MINIO_ACCESS_KEY $MINIO_SECRET_KEY
mc ls local/

# Create missing buckets
mc mb local/superadmin-uploads
mc mb local/superadmin-backups
mc mb local/superadmin-exports
mc mb local/superadmin-audit-logs

# Verify .env has the correct bucket name
# MINIO_BUCKET=superadmin-uploads
```

**Prevention:** Automate bucket creation as part of the setup process. The setup wizard (Step 6) verifies and creates required buckets.

---

### 7.2 Access Denied

**Symptom:**
```
S3 Error: Access Denied
```

**Cause:** The MinIO credentials in `.env` are incorrect or the user lacks the required permissions.

**Solution:**
```bash
# Test credentials
mc alias set test http://localhost:9000 $MINIO_ACCESS_KEY $MINIO_SECRET_KEY
mc ls test/
# If this fails, the credentials are wrong

# Check if the access key exists in MinIO
# Open MinIO Console at http://localhost:9001
# Navigate to Identity > Users and verify the access key

# Reset the root password if needed
docker exec superadmin-minio mc admin user add local newadmin newpassword
```

**Prevention:** Use dedicated service account credentials (not root credentials) for the application. Store them in secrets management.

---

### 7.3 Connection Timeout

**Symptom:**
```
Error: connect ETIMEDOUT 10.0.0.5:9000
```

**Cause:** MinIO is unreachable due to network configuration, firewall rules, or the MinIO service being down.

**Solution:**
```bash
# Check if MinIO is running
docker ps | grep minio
docker logs superadmin-minio

# Test connectivity
curl http://localhost:9000/minio/health/live
# Expected: HTTP 200

# If running in Docker, ensure the app can reach MinIO
# In docker-compose, use the service name: MINIO_ENDPOINT=minio
# On the host, use: MINIO_ENDPOINT=localhost

# Check firewall
sudo ufw status
sudo iptables -L -n | grep 9000
```

**Prevention:** Include MinIO in the Docker Compose health check dependencies. Monitor MinIO availability through the Prometheus metrics endpoint.

---

## 8. Docker Container Issues

### 8.1 Container Keeps Restarting

**Symptom:** `docker compose ps` shows the app container in a restart loop with status `Restarting (1)`.

**Cause:** The application crashes on startup due to missing environment variables, unavailable dependencies, or a code error.

**Solution:**
```bash
# Check logs for the crash reason
docker compose logs app --tail 100

# Common causes and fixes:

# 1. Environment variable validation failed
#    -> Check .env against .env.example

# 2. Database not ready
#    -> Ensure depends_on with service_healthy condition

# 3. Out of memory
docker stats --no-stream
#    -> Increase memory limit in docker-compose.yml

# 4. Port conflict
docker compose logs app | grep EADDRINUSE
#    -> Change APP_PORT or stop the conflicting process
```

**Prevention:** Use health checks with `depends_on` conditions in Docker Compose. Set `restart: unless-stopped` instead of `restart: always` during debugging.

---

### 8.2 Volume Permission Errors

**Symptom:**
```
EACCES: permission denied, open '/data/config.json'
```

**Cause:** The container runs as a non-root user (UID 1000) but the mounted volume was created by root.

**Solution:**
```bash
# Fix ownership of the volume directory
sudo chown -R 1000:1000 /path/to/volume

# Or set the user in docker-compose.yml
# services:
#   app:
#     user: "1000:1000"
```

**Prevention:** Always specify `user` in the Dockerfile or docker-compose.yml. Use named volumes instead of bind mounts when possible.

---

### 8.3 Network Connectivity Between Containers

**Symptom:** The app container cannot reach the database or Redis containers. Error messages show connection refused to the service name.

**Cause:** The containers are on different Docker networks, or the service name is misspelled.

**Solution:**
```bash
# List networks
docker network ls

# Inspect the network to see connected containers
docker network inspect superadmin_default

# Verify containers can reach each other
docker exec superadmin-app ping postgres
docker exec superadmin-app nslookup redis

# If on different networks, connect them
docker network connect superadmin_default superadmin-postgres
```

**Prevention:** Define all services in the same `docker-compose.yml` file. Use explicit network definitions when services need isolation.

---

## 9. Kubernetes Pod Crashes

### 9.1 CrashLoopBackOff

**Symptom:**
```
NAME                              READY   STATUS             RESTARTS   AGE
superadmin-api-7d8f6b5c4-x2k9l   0/1     CrashLoopBackOff   5          10m
```

**Cause:** The pod crashes immediately after starting. Kubernetes restarts it with exponential backoff.

**Solution:**
```bash
# Check pod logs
kubectl logs superadmin-api-7d8f6b5c4-x2k9l -n superadmin --previous

# Check events
kubectl describe pod superadmin-api-7d8f6b5c4-x2k9l -n superadmin

# Common causes:

# 1. Missing secrets
kubectl get secrets -n superadmin
# Ensure all required secrets exist

# 2. ConfigMap missing
kubectl get configmaps -n superadmin

# 3. Liveness probe failing too early
# Increase initialDelaySeconds in the deployment

# 4. Resource limits too low
kubectl top pod superadmin-api-7d8f6b5c4-x2k9l -n superadmin
# Increase resource limits if OOMKilled
```

**Prevention:** Use startup probes with generous timeouts for slow-starting applications. Set resource requests and limits based on observed usage. Run migrations as a pre-install/pre-upgrade Helm hook.

---

### 9.2 OOMKilled

**Symptom:**
```
Last State:   Terminated
Reason:       OOMKilled
Exit Code:    137
```

**Cause:** The container exceeded its memory limit and was killed by the kernel OOM killer.

**Solution:**
```bash
# Check current memory usage
kubectl top pods -n superadmin

# Increase the memory limit
# In values.yaml:
# api:
#   resources:
#     limits:
#       memory: 4Gi

# Upgrade the release
helm upgrade superadmin ./k8s/helm/superadmin -n superadmin -f values-production.yaml
```

Also investigate the application for memory leaks (see [Memory Leaks](#11-memory-leaks)).

**Prevention:** Set memory limits based on load testing results plus a 30% buffer. Monitor memory usage trends in Grafana to detect gradual increases before they cause OOMKill.

---

### 9.3 Pod Cannot Pull Image

**Symptom:**
```
Failed to pull image "registry.example.com/superadmin-api:v1.0.0": unauthorized
```

**Cause:** The Kubernetes node cannot authenticate to the container registry.

**Solution:**
```bash
# Create a registry secret
kubectl create secret docker-registry regcred \
  --docker-server=registry.example.com \
  --docker-username=deploy \
  --docker-password='your_token' \
  -n superadmin

# Reference it in the deployment
# spec:
#   template:
#     spec:
#       imagePullSecrets:
#         - name: regcred
```

**Prevention:** Use a service account with image pull secrets attached. For Harbor, create a robot account with pull-only permissions.

---

## 10. SSL Certificate Problems

### 10.1 Certificate Expired

**Symptom:** Browsers show "Your connection is not private" (NET::ERR_CERT_DATE_INVALID). API clients throw SSL verification errors.

**Cause:** The TLS certificate has expired and was not auto-renewed.

**Solution:**
```bash
# Check certificate expiry
echo | openssl s_client -connect {DOMAIN}:443 2>/dev/null | openssl x509 -noout -dates

# If using Certbot, renew manually
sudo certbot renew

# If using Traefik with Let's Encrypt, check the ACME storage
docker exec superadmin-traefik cat /letsencrypt/acme.json | jq '.letsencrypt.Certificates[0].domain'

# Restart Traefik to trigger renewal
docker compose restart traefik

# If using cert-manager in Kubernetes
kubectl describe certificate superadmin-tls -n superadmin
kubectl describe certificaterequest -n superadmin
```

**Prevention:** Configure auto-renewal with Certbot or cert-manager. Set up monitoring alerts that fire 14 days before certificate expiry:

```bash
# Add to cron — alert if cert expires within 14 days
echo | openssl s_client -connect {DOMAIN}:443 2>/dev/null | openssl x509 -noout -checkend 1209600
```

---

### 10.2 Mixed Content Errors

**Symptom:** The browser console shows "Mixed Content: The page was loaded over HTTPS but requested an insecure resource."

**Cause:** The application or frontend is making HTTP requests to internal services that should use HTTPS, or hardcoded `http://` URLs exist in the code or configuration.

**Solution:**
```bash
# Check .env for any http:// URLs that should be https://
grep -n "http://" .env

# Update APP_URL to use HTTPS
# APP_URL=https://{DOMAIN}

# Ensure MINIO_USE_SSL=true in production
```

**Prevention:** In production, set `APP_URL` with the `https://` scheme. Use relative URLs in the frontend wherever possible. Add a Content-Security-Policy header that blocks mixed content: `upgrade-insecure-requests`.

---

### 10.3 SSL Handshake Failure

**Symptom:**
```
Error: SSL routines:ssl3_get_server_certificate:certificate verify failed
```

**Cause:** The client does not trust the server's certificate chain. This occurs with self-signed certificates, corporate CAs, or when the intermediate certificate is missing.

**Solution:**
```bash
# Check the full certificate chain
openssl s_client -connect {DOMAIN}:443 -showcerts

# If intermediate cert is missing, add it to the chain file
cat server.crt intermediate.crt > fullchain.crt

# For Node.js clients connecting to internal services with custom CAs
# Set NODE_EXTRA_CA_CERTS=/path/to/ca.crt

# In Docker
# environment:
#   - NODE_EXTRA_CA_CERTS=/etc/ssl/certs/custom-ca.crt
# volumes:
#   - ./certs/custom-ca.crt:/etc/ssl/certs/custom-ca.crt:ro
```

**Prevention:** Always use the full certificate chain in production. Use Let's Encrypt or a trusted CA rather than self-signed certificates.

---

## 11. Memory Leaks

### 11.1 Gradual Memory Growth

**Symptom:** Application memory usage grows steadily over hours or days. Eventually the process is killed (OOMKilled in Kubernetes) or becomes extremely slow.

**Cause:** Common causes include event listeners that are registered but never removed, large objects held in closures, accumulating cache without eviction, or unfinished database connections.

**Solution:**
```bash
# Take a heap snapshot
kill -USR2 $(pgrep -f "node.*superadmin")
# This generates a heap snapshot file in the working directory

# Or use the V8 inspector
node --inspect=0.0.0.0:9229 dist/index.js

# In Chrome, open chrome://inspect and take a heap snapshot
# Compare two snapshots taken minutes apart to identify growing objects

# Quick mitigation: restart the process
# In Kubernetes, this happens automatically on OOMKill
# For Docker Compose:
docker compose restart app
```

**Prevention:**
- Close all database connections and Redis subscriptions in error handlers
- Remove event listeners in cleanup functions
- Set TTL on all Redis cache keys
- Use `WeakMap` and `WeakRef` for caches that should be garbage-collectible
- Run load tests with memory profiling before releases
- Set up Grafana alerts for memory usage exceeding 80% of the limit

---

### 11.2 High RSS Despite Low Heap

**Symptom:** The process RSS (Resident Set Size) is much larger than the V8 heap size reported by `process.memoryUsage()`.

**Cause:** Memory is allocated outside the V8 heap, typically by native modules (image processing, cryptographic operations) or Buffer allocations that are not garbage collected promptly.

**Solution:**
```bash
# Check memory breakdown
node -e "console.log(process.memoryUsage())"
# rss: total process memory
# heapUsed: V8 heap in use
# heapTotal: V8 heap allocated
# external: C++ objects bound to JavaScript
# arrayBuffers: ArrayBuffer and SharedArrayBuffer

# If external/arrayBuffers is large, check for unfreed Buffers
# Ensure streams are properly closed:
# stream.destroy() after use

# Limit Buffer pool size
# NODE_OPTIONS="--max-old-space-size=2048"
```

**Prevention:** Profile memory under load before each release. Use streaming for file operations rather than loading entire files into memory.

---

## 12. Performance Degradation

### 12.1 High API Latency

**Symptom:** API response times increase from the normal 30-50 ms to 500 ms or more. The `/api/v1/monitoring/api-metrics` endpoint confirms elevated p95 latency.

**Cause:** This can result from database query slowdowns, Redis latency, increased load, or inefficient code in a recent deployment.

**Solution:**
```bash
# Check which component is slow

# Database latency
curl -s http://localhost:3000/api/v1/monitoring/database-health \
  -H "Authorization: Bearer $ADMIN_TOKEN" | jq '.data'

# Redis latency
redis-cli -a $REDIS_PASSWORD --latency

# System resources
curl -s http://localhost:3000/api/v1/monitoring/system-health \
  -H "Authorization: Bearer $ADMIN_TOKEN" | jq '.data.system'

# Per-endpoint metrics
curl -s http://localhost:3000/api/v1/monitoring/api-metrics \
  -H "Authorization: Bearer $ADMIN_TOKEN" | jq '.data.endpoints | sort_by(-.avgResponseMs) | .[0:5]'

# If database is the bottleneck:
# - Check for missing indexes (see Section 1.4)
# - Check for long-running queries
# - Check connection pool saturation

# If Redis is the bottleneck:
# - Check memory usage and eviction rate
# - Check for slow commands: redis-cli -a $REDIS_PASSWORD SLOWLOG GET 10

# If CPU is the bottleneck:
# - Scale horizontally (add more pods)
# - Profile the application with Node.js --prof
```

**Prevention:** Establish baseline performance metrics. Set up Grafana alerts for latency exceeding 2x the baseline. Run load tests as part of the CI/CD pipeline.

---

### 12.2 High CPU Usage

**Symptom:** CPU usage is consistently above 80% on the application pods. Response times are elevated.

**Cause:** CPU-intensive operations (JSON parsing of large payloads, complex regex, bcrypt hashing, report generation) or a hot loop in the code.

**Solution:**
```bash
# Profile CPU usage
node --prof dist/index.js
# Generate the report
node --prof-process isolate-*.log

# In Kubernetes, check which pods are consuming CPU
kubectl top pods -n superadmin --sort-by=cpu

# Scale horizontally
kubectl scale deployment superadmin-api --replicas=6 -n superadmin

# Or let the HPA handle it automatically (if configured)
kubectl get hpa -n superadmin
```

**Prevention:** Move CPU-intensive tasks (report generation, bulk imports, virus scanning) to background job queues. Use worker threads for bcrypt hashing. Set appropriate HPA thresholds.

---

### 12.3 Disk Space Exhaustion

**Symptom:** The application fails with `ENOSPC: no space left on device`. Database writes fail. Log files cannot be written.

**Cause:** Log files, database WAL files, Docker images, or temporary files have filled the disk.

**Solution:**
```bash
# Check disk usage
df -h

# Find large files
du -sh /var/log/* | sort -rh | head -10
du -sh /var/lib/docker/* | sort -rh | head -10
du -sh /var/lib/postgresql/* | sort -rh | head -10

# Clean Docker resources
docker system prune -f
docker image prune -a -f

# Rotate logs
sudo journalctl --vacuum-size=500M

# Clean old database backups
mc ls local/superadmin-backups/ | head -n -7 | awk '{print $NF}' | \
  xargs -I {} mc rm local/superadmin-backups/{}

# Vacuum the database to reclaim space
sudo -u postgres psql -d superadmin_db -c "VACUUM FULL;"
```

**Prevention:** Configure log rotation (`max-size: 50m`, `max-file: 5` in Docker logging config). Set up disk space monitoring with alerts at 80% utilization. Automate old backup cleanup with the daily backup CronJob. Run `VACUUM` regularly on PostgreSQL.

---

## Additional Resources

- See `INSTALLATION.md` for initial setup troubleshooting
- See `DEPLOYMENT.md` for production operations
- See Section 36 of `SUPER_ADMIN_DOCUMENTATION.md` for performance optimization
- See Section 37 for security testing procedures
- See Section 47 for incident response procedures
