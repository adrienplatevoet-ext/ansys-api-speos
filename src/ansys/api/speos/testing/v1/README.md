# SpeosRPC Test Functionality

## Professional Solution Overview

### Purpose

The SpeosRPC Test Service provides a comprehensive automated testing framework for validating Speos optical simulation results. This service enables systematic verification of result files (XMP and Speos360 formats) against predefined XML templates, ensuring consistency, accuracy, and reliability of optical measurements across different versions and configurations.

### Solution Architecture

#### Core Functionality

The SpeosRPC test function accepts the following inputs:
- **Result Files**: XMP files (with future support for Speos360 format)
- **XML Template**: Validation schema defining expected measurements and properties
- **Reference Results** (optional): Baseline data for comparison

The function produces:
- **Viewer-Compatible Results**: Output formatted for display in the new viewer interface
- **Test Verdict**: Pass/fail status based on configurable comparison criteria with reference data

#### Template Compatibility

The service maintains backward compatibility between XML template versions:
- Current XML templates remain valid
- New XML templates created by SpeosRPC are compatible with existing templates
- The test function selectively extracts information available in the new viewer, ensuring template portability

Not all XML fields are processed during testing, as certain properties may not be available in the new viewer implementation. The service intelligently filters template fields based on viewer compatibility.

### Test Coverage Strategy

#### Automated Testing (Speos RPC in Speos One)

Complete combinatorial test coverage is achieved through automated Speos RPC tests within the Speos One environment. This ensures:
- All parameter combinations are validated
- Regression detection across software versions
- Consistent measurement validation

#### Manual Testing

Complementary manual test coverage validates:
- Visual display of measurements in the viewer
- Rule visualization and application
- Menu navigation and functionality
- Scale configuration and rendering
- User interface interactions

### Measurement Validation Process

Measurements undergo rigorous reliability validation via Speos RPC tests:

1. **Validation Phase**: Each measurement is verified for:
   - Name consistency and convention compliance
   - Value accuracy within defined tolerances
   - Unit correctness
   - Metadata completeness

2. **Promotion to Public APIs**: Once measurements pass validation:
   - They are promoted to public protobuf definitions
   - API contracts are established and versioned
   - Documentation is generated
   - Integration tests are created

This validation-first approach ensures that only reliable, well-tested measurements become part of the public API surface.

## API Usage

### Basic Test Execution

```python
from ansys.api.speos.testing.v1 import speos_rpc_test_pb2
from ansys.api.speos.testing.v1 import speos_rpc_test_pb2_grpc

# Create a test request
request = speos_rpc_test_pb2.TestRequest(
    result_file_uri="/path/to/result.xmp",
    xml_template=speos_rpc_test_pb2.XmlTemplate(
        template_content="<template>...</template>",
        version="1.0"
    ),
    test_config=speos_rpc_test_pb2.TestConfiguration(
        comparison_mode=speos_rpc_test_pb2.TestConfiguration.COMPARISON_MODE_TOLERANCE,
        tolerance_config=speos_rpc_test_pb2.ToleranceConfig(
            absolute_tolerance=0.01,
            relative_tolerance=0.05
        )
    )
)

# Execute test
response = stub.ExecuteTest(request)

# Check verdict
if response.verdict.status == speos_rpc_test_pb2.TestVerdict.TEST_STATUS_PASSED:
    print("Test passed!")
else:
    print(f"Test failed: {response.verdict.message}")
```

### Batch Test Execution

```python
# Create batch request for combinatorial testing
batch_request = speos_rpc_test_pb2.BatchTestRequest(
    test_requests=[test1, test2, test3],
    batch_config=speos_rpc_test_pb2.BatchConfiguration(
        parallel_workers=4,
        stop_on_failure=False
    )
)

batch_response = stub.ExecuteBatchTests(batch_request)
print(f"Passed: {batch_response.batch_summary.passed_tests}")
print(f"Failed: {batch_response.batch_summary.failed_tests}")
```

### Template Compatibility Validation

```python
# Validate template migration
compat_request = speos_rpc_test_pb2.TemplateCompatibilityRequest(
    current_template=current_xml_template,
    new_template=new_xml_template
)

compat_response = stub.ValidateTemplateCompatibility(compat_request)

if not compat_response.is_compatible:
    for issue in compat_response.compatibility_issues:
        print(f"{issue.severity}: {issue.description}")
```

### Measurement Reliability Validation

```python
# Validate measurements before promoting to public API
validation_request = speos_rpc_test_pb2.MeasurementValidationRequest(
    result_file_uri="/path/to/result.xmp",
    expected_measurements=[
        speos_rpc_test_pb2.MeasurementDefinition(
            name="total_flux",
            expected_type=speos_rpc_test_pb2.Measurement.MEASUREMENT_TYPE_INTENSITY,
            expected_unit="lm"
        )
    ],
    validation_criteria=speos_rpc_test_pb2.ValidationCriteria(
        strictness=speos_rpc_test_pb2.ValidationCriteria.STRICTNESS_LEVEL_STRICT,
        validate_names=True,
        validate_values=True,
        validate_units=True
    )
)

validation_response = stub.ValidateMeasurementReliability(validation_request)

if validation_response.is_valid:
    # Generate proto definitions from recommendations
    for recommendation in validation_response.proto_recommendations:
        print(f"{recommendation.field_name}: {recommendation.field_type}")
```

## Key Features

### 1. Comprehensive Test Execution
- Single test execution with detailed reporting
- Batch testing for combinatorial scenarios
- Configurable tolerance levels for numeric comparisons
- Multiple comparison modes (exact, tolerance-based, statistical)

### 2. Template Management
- XML template compatibility validation
- Field mapping between template versions
- Viewer compatibility detection
- Migration path recommendations

### 3. Measurement Validation
- Name, value, and unit verification
- Type checking and constraint validation
- Metadata completeness checks
- Proto definition generation recommendations

### 4. Result Analysis
- Detailed comparison results
- Field-by-field difference reporting
- Visual and numeric discrepancy detection
- Verdict generation with configurable criteria

### 5. Quality Assurance
- Automated regression testing
- Manual testing integration points
- Measurement reliability certification
- Public API promotion workflow

## Integration with Speos One

This testing framework integrates seamlessly with Speos One for automated testing:
- Execute tests as part of CI/CD pipelines
- Generate test reports in standardized formats
- Track measurement reliability over time
- Automate proto definition updates

## Benefits

1. **Quality Assurance**: Systematic validation of all optical measurements
2. **Regression Prevention**: Detect changes in measurement behavior across versions
3. **API Stability**: Only promote validated measurements to public APIs
4. **Backward Compatibility**: Maintain template compatibility across versions
5. **Automated Coverage**: Comprehensive combinatorial testing without manual effort
6. **Viewer Integration**: Ensure all displayed results are viewer-compatible

## Version History

- **v1**: Initial implementation with XMP support, XML template validation, and measurement reliability testing

## Future Enhancements

- Speos360 file format support
- Enhanced statistical comparison modes
- Advanced visualization of comparison results
- Integration with additional testing frameworks
