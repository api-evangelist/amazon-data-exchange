# Amazon Data Exchange

AWS Data Exchange makes it easy to find, subscribe to, and use third-party data in the cloud. Qualified data providers can publish data products consisting of data sets with versioned revisions and assets including S3 snapshots, Redshift data shares, API Gateway APIs, and Lake Formation permissions. Subscribers can find and subscribe to data products directly in the AWS Management Console and use the Data Exchange API to load data into Amazon S3 for analysis with AWS analytics and machine learning services.

## APIs

### AWS Data Exchange API

The AWS Data Exchange API enables programmatic access to find, subscribe to, and use third-party data products. It supports managing data sets, revisions, assets, jobs, and event actions for cloud-based data exchange workflows.

- **Base URL:** https://dataexchange.amazonaws.com
- **Documentation:** https://docs.aws.amazon.com/data-exchange/latest/apireference/
- **OpenAPI:** [openapi/amazon-data-exchange-openapi.yml](openapi/amazon-data-exchange-openapi.yml)

**Tags:** Data Marketplace, Data Products, Subscriptions, Analytics

## Artifacts

| Type | Count | Location |
|------|-------|----------|
| OpenAPI Specs | 1 | [openapi/](openapi/) |
| JSON Schemas | 22 | [json-schema/](json-schema/) |
| JSON Structures | 22 | [json-structure/](json-structure/) |
| Examples | 22 | [examples/](examples/) |
| JSON-LD Contexts | 1 | [json-ld/](json-ld/) |
| Spectral Rules | 1 | [rules/](rules/) |
| Vocabulary | 1 | [vocabulary/](vocabulary/) |
| Naftiko Capabilities | 2 | [capabilities/](capabilities/) |

## Features

- **Data Set Management** — Create, update, and manage data sets containing versioned collections of data available for subscription and distribution in the marketplace.
- **Revision Publishing** — Organize data into versioned revisions with comments, then finalize and publish them to make data available to subscribers automatically.
- **Multi-Format Asset Support** — Support for S3 snapshots, Redshift data shares, API Gateway APIs, Lake Formation permissions, and S3 data access as asset types.
- **Import and Export Jobs** — Asynchronous import/export jobs for transferring data between external sources (S3, Redshift) and Data Exchange revisions at scale.
- **Event-Driven Delivery** — Configurable event actions that automatically export revision data to S3 when a new revision is published, eliminating manual downloads.
- **AWS Marketplace Integration** — Seamlessly list and sell data products in AWS Marketplace with built-in billing, subscription management, and entitlement enforcement.
- **Fine-Grained Access Control** — Control access to data products using AWS IAM policies and resource-level permissions with ARN-based resource identification.

## Use Cases

- **Third-Party Data Acquisition** — Subscribe to curated third-party datasets from financial data providers, healthcare data aggregators, weather services, and market research firms.
- **Data Product Monetization** — Publish and sell proprietary datasets to other AWS customers via the marketplace with automated billing and subscription management.
- **Automated Data Pipelines** — Configure event actions to automatically deliver new data revisions to S3, enabling downstream analytics pipelines to process fresh data.
- **ML Training Data** — Access high-quality labeled datasets and specialized data products from Data Exchange to train and improve machine learning models.
- **Regulatory Compliance Data** — Subscribe to compliance reference data including sanctions lists, legal entity identifiers, and regulatory taxonomies via Data Exchange.

## Integrations

- **Amazon S3** — Primary storage integration for Data Exchange assets — used as both source for imports and destination for exports via job operations.
- **Amazon Redshift** — Share Redshift tables and views directly through Data Exchange without copying data, enabling live query access for subscribers.
- **AWS Lake Formation** — Distribute Lake Formation data permissions through Data Exchange, giving subscribers governed access to data lake resources.
- **Amazon API Gateway** — Expose API-based data products through Data Exchange, allowing subscribers to call APIs for real-time data access.
- **AWS Glue** — Catalog and transform Data Exchange S3 datasets using AWS Glue for ETL and data lake integration workflows.
- **Amazon Athena** — Query Data Exchange S3 snapshot data directly with SQL using Athena for serverless analytics on subscribed datasets.
- **Amazon SageMaker** — Use third-party datasets from Data Exchange as training data for machine learning models in Amazon SageMaker.

## Resources

- [Portal](https://aws.amazon.com/data-exchange/)
- [Documentation](https://docs.aws.amazon.com/data-exchange/)
- [Getting Started](https://aws.amazon.com/data-exchange/getting-started/)
- [Pricing](https://aws.amazon.com/data-exchange/pricing/)
- [FAQ](https://aws.amazon.com/data-exchange/faqs/)
- [Blog](https://aws.amazon.com/blogs/big-data/category/analytics/aws-data-exchange/)
- [GitHub Organization](https://github.com/aws)
- [Console](https://console.aws.amazon.com/dataexchange/)
- [Support](https://aws.amazon.com/premiumsupport/)
- [Terms of Service](https://aws.amazon.com/service-terms/)
- [Privacy Policy](https://aws.amazon.com/privacy/)

## Maintainers

- **Kin Lane** — kin@apievangelist.com
