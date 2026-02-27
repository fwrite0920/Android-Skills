# Changelog

## [1.1.1] - 2026-02-27

### Fixes
- **deep_performance_tuning**: Correct lifecycleScope owner reference
- **dependency_injection_mastery**: Add EntryPoint interface and fix service accessor
- **legacy_rapid_expansion**: Add missing parameters to navigation methods
- **navigation_patterns**: Move comment out of JSON code block
- **sdk_development**: Use interface for API surface and fix markdown nesting
- **supply_chain_security**: Move comment out of JSON code block
- **tech_stack_migration**: Use @ApplicationContext for DI bridge module
- **ui_ux_engineering**: Add missing activity variable declaration

### Documentation
- **coding_style_conventions**: Rename Code Review Checklist to Quick Checklist
- Update skill count to 17 and generalize skills-root paths in guide

## [1.1.0] - 2026-02-27

### Features
- **observability_first**: Add comprehensive code examples and implementation patterns including Firebase Performance, OkHttp monitoring, unified analytics tracker, crash context management, and CI gate configuration
- **supply_chain_security**: Add comprehensive security implementation examples including Gradle dependency verification, OWASP Dependency-Check, Renovate/Dependabot configuration, and full security CI pipeline
- **sdk_development**: Add new SDK/Library development skill covering API design, module structure, Maven Central publishing, binary compatibility validation, versioning strategy, and Dokka documentation

### Fixes
- Improve code quality and thread safety in crash_monitoring and data_layer_mastery skills
- Add @Volatile annotations to ANRWatchDog for proper thread visibility
- Fix force unwrap and connection leak issues

### Documentation
- Update all documentation to reflect 17 skills (previously 16)
- Add sdk_development to skill index and scenario router
- Update marketplace.json to version 1.1.0

## [1.0.0] - 2026-02-27

### Features
- Initial release with 16 Android development skills
- Skill index and navigation system
- Support for Claude Code plugin marketplace installation via `npx skills add`
