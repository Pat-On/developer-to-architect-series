# Netflix Ribbon Load Balancer Code and Configuration

```java
    @Override
    public boolean reserveInventory(InventoryReservation inventoryReservation) throws IOException {
        // load balancer
        ServiceInstance instance = getLoadBalancer().choose(ServiceID.InventorySvc.toString());
        if (instance == null) {
            logger.error("Unable to reserve inventory, as inventory service is not available");
            throw new IOException("Inventory service is not available");
        }

        // builder and then normal call
        StringBuilder url = new StringBuilder()
                .append("http://").append(instance.getHost())
                .append(":").append(instance.getPort())
                .append(AppConfig.INVENTORY_RESOURCE_PATH)
                .append(AppConfig.INVENTORY_RESERVATION_PATH);
        String invResJson = (new Gson()).toJson(inventoryReservation);
        RequestBody body = RequestBody.create(invResJson, okhttp3.MediaType.parse("application/json; charset=utf-8"));
        Request.Builder requestBuilder = new Request.Builder()
                .url(url.toString())
                .post(body);
        String authHeader = OrderServiceImpl.getThreadLocal().get();
        requestBuilder.addHeader("Authorization", authHeader);
        Span invRemoteCallSpan = tracer.buildSpan("reserveInventoryRemote").asChildOf(tracer.activeSpan()).start();
        tracer.inject(invRemoteCallSpan.context(),
                    Format.Builtin.HTTP_HEADERS,
                    new RequestBuilderCarrier(requestBuilder));
        Request request = requestBuilder.build();
        try (Response response = client.newCall(request).execute()) {
            if (response.code() == 200) {
                logger.debug("Reserved inventory successfully; context={}", inventoryReservation);
                return true;
            }
            logger.debug("Unable to reserve inventory, got error; errorCode={}, message={} context={}",
                        response.code(), response.message(), inventoryReservation);
            return false;
        } catch (IOException e) {
            logger.error("Error calling InventorySvc while reserving inventory; context={}", inventoryReservation);
            logger.error(e.getMessage(), e);
            throw e;
        }
        finally {
            invRemoteCallSpan.finish();
        }
    }

```