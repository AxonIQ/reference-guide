# Minor Releases

Any patch release made for an Axon project aims to resolve bugs.
This page provides an overview of patch releases for the Axon Tracing Extension.

## Release 4.3

### Release 4.3.1

* The tag for the query payload was wrong.
  We give credits to contributor `Sam-Kruglov` for noting the issue, which we resolved in [this](https://github.com/AxonFramework/extension-tracing/commit/72c8b15fec144c62fe6115d4993d60ab93ecee07) commit.

* In some Tracing UI's, camel-cased tag names showed up without the separation.
  This discrepancy is marked in issue [#45](https://github.com/AxonFramework/extension-tracing/issues/45) and resolved by using a dash notation instead of camel casing.
 