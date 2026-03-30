## .NET .editorconfig Example
```
root = true

[*]
indent_style = space
end_of_line = crlf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true

[*.cs]
indent_size = 4

# Organize usings
dotnet_sort_system_directives_first = true
dotnet_separate_import_directive_groups = false

# this. preferences
dotnet_style_qualification_for_field = false
dotnet_style_qualification_for_property = false
dotnet_style_qualification_for_method = false
dotnet_style_qualification_for_event = false

# Language keywords over framework types
dotnet_style_predefined_type_for_locals_parameters_members = true
dotnet_style_predefined_type_for_member_access = true

# Modifier preferences
dotnet_style_require_accessibility_modifiers = for_non_interface_members
dotnet_style_readonly_field = true
csharp_preferred_modifier_order = public, private, protected, internal, file, static, extern, new, virtual, abstract, sealed, override, readonly, unsafe, required, volatile, async

# Expression preferences
dotnet_style_object_initializer = true
dotnet_style_collection_initializer = true
dotnet_style_prefer_auto_properties = true
dotnet_style_explicit_tuple_names = true
dotnet_style_prefer_inferred_tuple_names = true
dotnet_style_prefer_inferred_anonymous_type_member_names = true
dotnet_style_prefer_is_null_check_over_reference_equality_method = true
dotnet_style_coalesce_expression = true
dotnet_style_null_propagation = true

# Primary constructor preference (IDE0290)
csharp_style_prefer_primary_constructors = true
dotnet_diagnostic.IDE0290.severity = suggestion

# C# style preferences
csharp_style_var_for_built_in_types = true
csharp_style_var_when_type_is_apparent = true
csharp_style_var_elsewhere = true
csharp_style_expression_bodied_methods = when_on_single_line
csharp_style_expression_bodied_properties = when_on_single_line
csharp_style_expression_bodied_constructors = false
csharp_style_pattern_matching_over_is_with_cast_check = true
csharp_style_pattern_matching_over_as_with_null_check = true
csharp_style_throw_expression = true
csharp_style_conditional_delegate_call = true
csharp_prefer_braces = true
csharp_style_namespace_declarations = file_scoped

# Newline preferences
csharp_new_line_before_open_brace = all
csharp_new_line_before_else = true
csharp_new_line_before_catch = true
csharp_new_line_before_finally = true
csharp_new_line_before_members_in_object_initializers = true
csharp_new_line_before_members_in_anonymous_types = true
csharp_new_line_between_query_expression_clauses = true

# Indentation preferences
csharp_indent_case_contents = true
csharp_indent_switch_labels = true
csharp_indent_labels = flush_left

# Space preferences
csharp_space_after_cast = false
csharp_space_after_keywords_in_control_flow_statements = true
csharp_space_between_method_call_parameter_list_parentheses = false
csharp_space_between_method_declaration_parameter_list_parentheses = false
csharp_space_before_colon_in_inheritance_clause = true
csharp_space_after_colon_in_inheritance_clause = true
csharp_space_around_binary_operators = before_and_after
csharp_space_between_method_declaration_empty_parameter_list_parentheses = false
csharp_space_between_method_call_name_and_opening_parenthesis = false
csharp_space_between_method_call_empty_parameter_list_parentheses = false

# Wrapping preferences
csharp_preserve_single_line_statements = false
csharp_preserve_single_line_blocks = true

# Naming conventions — PascalCase for types, methods, properties
dotnet_naming_rule.types_and_namespaces.symbols = types_and_namespaces
dotnet_naming_rule.types_and_namespaces.style = pascal_case_style
dotnet_naming_rule.types_and_namespaces.severity = warning
dotnet_naming_symbols.types_and_namespaces.applicable_kinds = namespace, class, struct, interface, enum
dotnet_naming_symbols.types_and_namespaces.applicable_accessibilities = *

dotnet_naming_rule.methods_and_properties.symbols = methods_and_properties
dotnet_naming_rule.methods_and_properties.style = pascal_case_style
dotnet_naming_rule.methods_and_properties.severity = warning
dotnet_naming_symbols.methods_and_properties.applicable_kinds = method, property, event
dotnet_naming_symbols.methods_and_properties.applicable_accessibilities = *

# Interfaces prefixed with I
dotnet_naming_rule.interfaces.symbols = interfaces
dotnet_naming_rule.interfaces.style = interface_style
dotnet_naming_rule.interfaces.severity = warning
dotnet_naming_symbols.interfaces.applicable_kinds = interface
dotnet_naming_symbols.interfaces.applicable_accessibilities = *
dotnet_naming_style.interface_style.required_prefix = I
dotnet_naming_style.interface_style.capitalization = pascal_case

# Private fields prefixed with _
dotnet_naming_rule.private_fields.symbols = private_fields
dotnet_naming_rule.private_fields.style = private_field_style
dotnet_naming_rule.private_fields.severity = warning
dotnet_naming_symbols.private_fields.applicable_kinds = field
dotnet_naming_symbols.private_fields.applicable_accessibilities = private, private_protected
dotnet_naming_style.private_field_style.required_prefix = _
dotnet_naming_style.private_field_style.capitalization = camel_case

# Local variables and parameters: camelCase
dotnet_naming_rule.locals_and_parameters.symbols = locals_and_parameters
dotnet_naming_rule.locals_and_parameters.style = camel_case_style
dotnet_naming_rule.locals_and_parameters.severity = warning
dotnet_naming_symbols.locals_and_parameters.applicable_kinds = local, parameter
dotnet_naming_style.camel_case_style.capitalization = camel_case
dotnet_naming_style.pascal_case_style.capitalization = pascal_case

[*.{json,yaml,yml}]
indent_size = 2

[*.md]
trim_trailing_whitespace = false

[*.{csproj,props,targets}]
indent_size = 2
```