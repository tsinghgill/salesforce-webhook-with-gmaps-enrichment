# Salesforce to Webhook Pipeline with Data Enrichment

## Pipeline Description

### Overview
This pipeline captures changes in Salesforce via the Salesforce Source Connector, enriches the data using the Google Maps API, and forwards the enriched data to a specified webhook destination. It integrates CRUD operations captured from Salesforce, data enrichment via an external API, and robust error handling mechanisms.

### Salesforce Source Connector
The Salesforce Source Connector is set up to listen to specific Salesforce push topics, particularly "AccountUpdates". This configuration allows it to capture real-time CRUD operations on Salesforce records securely via OAuth.

- **Client ID and Secret**: Used for Salesforce API authentication.
- **Environment**: Points to the specific Salesforce development environment.
- **Username and Password**: Credentials for Salesforce access.
- **Security Token**: Adds a layer of security for Salesforce client verification.
- **Push Topics**: Enables the capture of changes to Salesforce objects as specified by the topic configuration.

### Data Enrichment with webhook.http Processor
Data fetched from Salesforce is passed to the `webhook.http` processor, which constructs a URL to call the Google Maps Geocoding API, inserting address components dynamically from the Salesforce data into the API request. The processor safely encodes the URL parameters and retrieves geographic coordinates and other address details.

- **Request URL**: Dynamically constructed using Salesforce record address fields.
- **Response Handling**: API responses are JSON-encoded and stored in `.Payload.After.enrichedAddress` with HTTP status in `.Metadata["http_status"]`.

### JSON Decoding
After data enrichment, the `decode-json` processor decodes the JSON response stored in `.Payload.After.enrichedAddress`, converting it back into structured data for downstream usage.

### HTTP Destination/Sink Connector
Processed data is sent to an external webhook defined in the HTTP destination/sink connector settings, enabling integration with various web services.

- **URL**: Endpoint for the processed data.

### Dead-Letter Queue
Ensures robust error handling by capturing records that fail processing, stored at the specified path for analysis or reprocessing.

- **Path**: Storage path for dead-letter records.
- **Window Size and Nack Threshold**: Configurations for handling failed records.

## Getting It Up and Running

### 1. Install Conduit
Begin by installing Conduit. Detailed instructions are available on the [Conduit Getting Started page](https://conduit.io/docs/introduction/getting-started/).

### 2. Set up Connectors
Download and save the latest standalone connector binaries to the connectors directory for Conduit to reference:

- [Salesforce Connector Binary](https://github.com/conduitio-labs/conduit-connector-salesforce/releases)
- [HTTP Connector Binary](https://github.com/conduitio-labs/conduit-connector-http/releases)

For more details on how Conduit references connectors, visit [Conduit Connectors Setup](https://conduit.io/docs/connectors/getting-started).

### 3. Setup Your Google Maps API
Configure your Google Maps API credentials and ensure the Salesforce data fields are correctly mapped to feed the API:

- [Google Maps Geocoding API Overview](https://developers.google.com/maps/documentation/geocoding/overview)
- [Salesforce Push Topics](https://developer.salesforce.com/docs/atlas.en-us.api_streaming.meta/api_streaming/code_sample_interactive_vfp_create_pushtopic.htm)

### Done
Happy streaming! Once set up, your pipeline will automatically process Salesforce data through Google Maps enrichment and forward it to your specified webhook.
