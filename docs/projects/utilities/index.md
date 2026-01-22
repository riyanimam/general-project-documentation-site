# Utility Projects

This section covers utility projects for automation, data collection, and project management.

## Project Overview

| Project | Description | Language | Primary Use |
|---------|-------------|----------|-------------|
| [AWS Environment Cleanup Lambda](aws-environment-cleanup-lambda.md) | Automated cleanup of AWS resources | Python | AWS maintenance |
| [Georgia Legislation Webcrawler](georgia-legislation-webcrawler.md) | Web scraper for legislation data | Python/TypeScript | Data collection |
| [Kanban Board Template](not-confusing-kanban-board-template.md) | GitHub project template | YAML | Project management |

## Categories

### Automation

Tools for automating repetitive tasks and maintenance operations.

<div class="grid cards" markdown>

-   :material-broom:{ .lg .middle } **AWS Environment Cleanup Lambda**

    ---

    Automated cleanup of unused AWS resources including Lambda functions,
    IAM roles, QuickSight assets, and more.

    [:octicons-arrow-right-24: Learn more](aws-environment-cleanup-lambda.md)

</div>

### Data Collection

Projects focused on gathering and processing data from various sources.

<div class="grid cards" markdown>

-   :material-spider:{ .lg .middle } **Georgia Legislation Webcrawler**

    ---

    Full-stack application for scraping, storing, and analyzing Georgia
    state legislation data.

    [:octicons-arrow-right-24: Learn more](georgia-legislation-webcrawler.md)

</div>

### Templates

Ready-to-use templates for project management and organization.

<div class="grid cards" markdown>

-   :material-view-dashboard:{ .lg .middle } **Kanban Board Template**

    ---

    Simple, intuitive GitHub project template for organizing work
    without the confusion of complex project boards.

    [:octicons-arrow-right-24: Learn more](not-confusing-kanban-board-template.md)

</div>

## Common Patterns

### Python Best Practices

All Python-based utilities follow consistent patterns:

```python
# Structured logging
import logging

logger = logging.getLogger(__name__)
logger.setLevel(logging.INFO)

# Type hints for clarity
def process_resource(resource_id: str, dry_run: bool = True) -> dict:
    """Process a resource with optional dry-run mode."""
    ...

# Environment-based configuration
import os

DRY_RUN = os.getenv("DRY_RUN", "true").lower() == "true"
```

### Error Handling

Consistent error handling across utilities:

```python
from dataclasses import dataclass
from typing import Optional

@dataclass
class Result:
    success: bool
    message: str
    resource_id: Optional[str] = None
    error: Optional[Exception] = None

def safe_operation(func):
    """Decorator for safe execution with result tracking."""
    def wrapper(*args, **kwargs):
        try:
            result = func(*args, **kwargs)
            return Result(success=True, message="Success", resource_id=result)
        except Exception as e:
            return Result(success=False, message=str(e), error=e)
    return wrapper
```

## Quick Links

- [AWS Environment Cleanup Lambda :material-arrow-right:](aws-environment-cleanup-lambda.md)
- [Georgia Legislation Webcrawler :material-arrow-right:](georgia-legislation-webcrawler.md)
- [Kanban Board Template :material-arrow-right:](not-confusing-kanban-board-template.md)
