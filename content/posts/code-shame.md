+++
date = '2013-01-03'
title = 'Code Shame'
+++

Naturally when I write C I'm reasonably careful about a number of things.
Because at runtime it's a free-for-all, I take care to notice buffer
overruns, uninitialized memory, and in particular the possible values
assigned to a pointer.

I'll back up a second, because unfortunately I recently managed to make
a bit of a bollocks of the last point, which I only noticed
when I came back to change the code. This is something I know inside-out,
so the point of this post is to shame me into never making the same mistake
again.

```diff
--- a/src/resource_handler.c
+++ b/src/resource_handler.c
@@ -27,16 +27,18 @@ uint16_t
service_request(struct request *req, char *rtrv_buf, const size_t len)
{
 
-    if (req->method == MGET) {
-        if (cache_rtrv(req->uri, rtrv_buf, len) == 0) {
-            return ROK;
-        } else {
-            log_ln(DEBUG, "'%s' not found in cache\n", req->uri);
-            return RNOTFOUND;
+    if (req != NULL) {
+        if (req->method == MGET) {
+            if (cache_rtrv(req->uri, rtrv_buf, len) == 0) {
+                return ROK;
+            } else {
+                log_ln(DEBUG, "'%s' not found in cache\n", req->uri);
+                return RNOTFOUND;
+            }
+        } else if (req->method == MUNKNOWN || req->method == MPOST) {
+            log_ln(ERROR, "request method not implemented: 0x%x\n", req->method
+            return RNOTIMPLEMENTED;
         }
-    } else if (req->method == MUNKNOWN) {
-        log_ln(ERROR, "request method not implemented: 0x%x\n", req->method);
-        return RNOTIMPLEMENTED;
     }
 
     return RINTERNAL;
```

In the original code I'm deferencing a field in a struct (`req->method`)
without first finding out whether the struct had been assigned.

To fix this I added a check for a `NULL` before trying to dereference
the field (and as this is an HTTP server, prepare to send an HTTP 500
response to the client if there's a problem).
