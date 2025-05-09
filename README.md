# ht_data_api

![coverage: 100%](https://img.shields.io/badge/coverage-100-green)
[![style: very good analysis](https://img.shields.io/badge/style-very_good_analysis-B22C89.svg)](https://pub.dev/packages/very_good_analysis)
[![License: PolyForm Free Trial](https://img.shields.io/badge/License-PolyForm%20Free%20Trial-blue)](https://polyformproject.org/licenses/free-trial/1.0.0)

A generic Dart package providing a concrete implementation of the `HtDataClient<T>` abstract class for interacting with data resource endpoints via HTTP. It leverages the `ht_http_client` package for underlying HTTP communication and error handling.

## Getting Started

Add this package to your `pubspec.yaml` dependencies:

```yaml
dependencies:
  ht_data_api:
    git:
      url: https://github.com/headlines-toolkit/ht-data-api.git
      # ref: <specific_tag_or_commit> # Optional: Pin to a specific version
  ht_http_client:
    git:
      url: https://github.com/headlines-toolkit/ht-http-client.git
      # ref: <specific_tag_or_commit>
```

Then run `dart pub get` or `flutter pub get`.

## Features

*   Provides a concrete implementation of the `HtDataClient<T>` abstract class.
*   Implements data access methods (`create`, `read`, `update`) returning `Future<SuccessApiResponse<T>>`.
*   Implements list retrieval methods (`readAll`, `readAllByQuery`) returning `Future<SuccessApiResponse<PaginatedResponse<T>>>`.
*   Implements `delete` returning `Future<void>`.
*   Requires an instance of `HtHttpClient` for making HTTP requests.
*   Configurable with the `modelName` (identifying the resource) and `fromJson`/`toJson` functions for the specific model `T`.
*   Propagates `HtHttpException` errors from the underlying `HtHttpClient`.
*   Includes comprehensive unit tests with 100% coverage.

## Usage

1.  **Define your Model:** Create a class representing the data structure for your API resource. Include `fromJson` and `toJson` methods/functions.

    ```dart
    class MyModel {
      const MyModel({required this.id, required this.name});

      final String id;
      final String name;

      // Factory constructor for deserialization
      factory MyModel.fromJson(Map<String, dynamic> json) {
        return MyModel(
          id: json['id'] as String,
          name: json['name'] as String,
        );
      }

      // Method for serialization
      Map<String, dynamic> toJson() {
        return {
          'id': id,
          'name': name,
        };
      }
    }
    ```

2.  **Instantiate `HtHttpClient`:** Set up your HTTP client (refer to `ht_http_client` documentation).

    ```dart
    // Example setup (replace with your actual implementation)
    Future<String?> _myTokenProvider() async => 'your_auth_token';

    final httpClient = HtHttpClient(
      baseUrl: 'https://api.yourapp.com/v1',
      tokenProvider: _myTokenProvider,
    );
    ```

3.  **Instantiate `HtDataApi`:** Create an instance specific to your model, providing the `modelName` used in the unified API endpoint.

    ```dart
    final myModelApi = HtDataApi<MyModel>(
      httpClient: httpClient,
      modelName: 'my-models', // The name identifying this resource in the API
      fromJson: MyModel.fromJson, // Reference to your fromJson factory/function
      toJson: (model) => model.toJson(), // Reference to your toJson method/function
    );
    ```

4.  **Perform CRUD Operations:**

    ```dart
    try {
      // Create
      final newModelData = MyModel(id: '', name: 'New Item'); // ID might be ignored by API
      final createResponse = await myModelApi.create(newModelData);
      final createdModel = createResponse.data; // Access data from envelope
      print('Created: ${createdModel.id}');

      // Read All
      final readAllResponse = await myModelApi.readAll();
      final allModels = readAllResponse.data.items; // Access items from paginated data
      print('Found ${allModels.length} models.');
      print('Has more: ${readAllResponse.data.hasMore}');

      // Read All by Query
      final queryResponse = await myModelApi.readAllByQuery({
        'name': 'New Item',
        'limit': 1,
      });
      final queryResults = queryResponse.data.items; // Access items
      print('Found ${queryResults.length} models matching query.');
      if (queryResults.isNotEmpty) {
        print('First query result: ${queryResults.first.name}');
      }

      // Read One
      if (allModels.isNotEmpty) {
        final firstModelId = allModels.first.id;
        final readResponse = await myModelApi.read(firstModelId);
        final fetchedModel = readResponse.data; // Access data from envelope
        print('Fetched: ${fetchedModel.name}');

        // Update
        final updatedData = MyModel(id: firstModelId, name: 'Updated Name');
        final updateResponse = await myModelApi.update(firstModelId, updatedData);
        final updatedModel = updateResponse.data; // Access data from envelope
        print('Updated: ${updatedModel.name}');

        // Delete (no change in return type)
        await myModelApi.delete(firstModelId);
        print('Deleted model with ID: $firstModelId');
      }
    } on HtHttpException catch (e) {
      // Handle specific HTTP errors from ht_http_client
      print('API Error: $e');
      if (e is NotFoundException) {
        // Handle 404 specifically
      }
      // ... other specific exception types
    } catch (e) {
      // Handle other potential errors (e.g., FormatException during deserialization)
      print('An unexpected error occurred: $e');
    }
    ```

## License

This package is licensed under the [PolyForm Free Trial 1.0.0](LICENSE). Please review the terms before use.
