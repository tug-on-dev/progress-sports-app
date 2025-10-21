# API Documentation

> **📖 Complete Version**: For full API documentation with detailed examples, see [French API Documentation](../../fr/api/api-documentation.md)

## Quick Reference

### Base URL
```
Local: http://localhost:8080/web/
Production: http://[load-balancer-url]/web/
```

### Available Endpoints

| Endpoint | Method | Description | 4GL Program |
|----------|--------|-------------|-------------|
| `/web/customer-data.p` | GET | Customer data in JSON | customer-data.p |
| `/web/state-data.p` | GET | State data in JSON | state-data.p |
| `/web/customer-view.html` | GET | Customer grid view | customer-view.html |
| `/web/state-view.html` | GET | State grid view | state-view.html |
| `/web/about.html` | GET | About page | about.html |

## Customer Data API

### Endpoint
```
GET /web/customer-data.p
```

### Response Format
```json
{
  "dsCustomer": {
    "ttCustomer": [
      {
        "CustNum": 1,
        "Country": "USA",
        "Name": "LIFT LINE SKIING",
        "Address": "276 North Drive",
        "Address2": ""
      }
    ]
  }
}
```

### Example Usage

**cURL**:
```bash
curl -X GET http://localhost:8080/web/customer-data.p \
  -H "Accept: application/json"
```

**JavaScript**:
```javascript
fetch('/web/customer-data.p')
    .then(response => response.json())
    .then(data => {
        console.log('Customers:', data.dsCustomer.ttCustomer);
    });
```

## State Data API

### Endpoint
```
GET /web/state-data.p
```

### Response Format
```json
{
  "dsState": {
    "ttState": [
      {
        "State": "AL",
        "StateName": "Alabama",
        "Region": "South"
      }
    ]
  }
}
```

## Testing

### Automated Tests
```bash
# Test all APIs
./test.sh localhost:8080

# Individual endpoint test
curl -s http://localhost:8080/web/customer-data.p | jq .
```

### HTTP Status Codes

| Code | Description | Meaning |
|------|-------------|---------|
| 200 | OK | Request successful |
| 404 | Not Found | Endpoint doesn't exist |
| 500 | Internal Server Error | 4GL program error |
| 502 | Bad Gateway | PASOE unavailable |
| 503 | Service Unavailable | System overload |

## Headers

### Request Headers
```
Accept: application/json
Content-Type: application/json
```

### Response Headers
```
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Cache-Control: no-cache, no-store, must-revalidate
```

## Frontend Integration

### Kendo UI DataSource
```javascript
var dataSource = new kendo.data.DataSource({
    transport: {
        read: {
            url: '/web/customer-data.p',
            dataType: "json"
        }
    },
    schema: {
        data: "dsCustomer.ttCustomer"
    },
    pageSize: 20
});
```

### Grid Configuration
```javascript
$("#grid").kendoGrid({
    dataSource: dataSource,
    navigatable: true,
    filterable: true,
    sortable: true,
    pageable: true,
    columns: [
        { field: "CustNum", title: "Customer #" },
        { field: "Name", title: "Name" },
        { field: "Country", title: "Country" },
        { field: "Address", title: "Address" }
    ]
});
```

## Error Handling

### Error Format
```json
{
  "error": {
    "code": 500,
    "message": "Internal server error",
    "details": "Database connection failed"
  }
}
```

### Global Error Handler
```javascript
$(document).ajaxError(function(event, xhr, settings, error) {
    console.error("API Error:", xhr.status, error);
    kendo.alert("Communication error with server");
});
```

## Performance

### Limits
- **Concurrent connections**: 200 per PASOE agent
- **Request timeout**: 15 seconds
- **Max response size**: 10 MB (configurable)

### Optimizations
- nginx caching for static resources
- gzip compression enabled
- Keep-alive connections
- Database connection pooling

---

**📚 For complete documentation including:**
- Detailed request/response examples
- Advanced filtering and pagination
- Security configuration
- Performance tuning
- Monitoring and debugging

**See the [complete French documentation](../../fr/api/api-documentation.md)**
