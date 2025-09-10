# Coding Standards

## General Principles

### Code Quality Standards
- Write clean, readable, and maintainable code
- Follow DRY (Don't Repeat Yourself) principle
- Use meaningful variable and function names
- Keep functions small and focused
- Comment complex logic and algorithms

### Consistency
- Follow established patterns in the codebase
- Use consistent naming conventions
- Maintain consistent indentation and formatting

## Language-Specific Standards

### Python (if applicable)
<!-- TODO: Python-specific standards -->

#### Style Guide
- Follow PEP 8 style guide
- Use type hints where appropriate
- Maximum line length: 88 characters

#### Naming Conventions
```python
# Variables and functions: snake_case
user_name = "john_doe"
def calculate_distance():
    pass

# Classes: PascalCase
class DataProcessor:
    pass

# Constants: UPPER_SNAKE_CASE
MAX_RETRY_ATTEMPTS = 3
```

#### Code Examples
```python
# Good
def process_nasa_data(data: List[Dict]) -> Dict:
    """Process NASA API data and return summary statistics."""
    if not data:
        raise ValueError("Data cannot be empty")
    
    return {
        "total_records": len(data),
        "processed_at": datetime.now()
    }

# Avoid
def proc_data(d):
    return {"total": len(d), "time": datetime.now()}
```

### JavaScript/TypeScript (if applicable)
<!-- TODO: JavaScript/TypeScript standards -->

#### Style Guide
- Use ESLint with recommended rules
- Use Prettier for code formatting
- Prefer const/let over var

#### Naming Conventions
```javascript
// Variables and functions: camelCase
const userName = 'johnDoe';
function calculateDistance() {}

// Classes: PascalCase
class DataProcessor {}

// Constants: UPPER_SNAKE_CASE
const MAX_RETRY_ATTEMPTS = 3;
```

### HTML/CSS (if applicable)
<!-- TODO: HTML/CSS standards -->

#### HTML Standards
- Use semantic HTML elements
- Include alt text for images
- Use proper heading hierarchy

#### CSS Standards
- Use BEM methodology for class naming
- Group related properties
- Use CSS custom properties for repeated values

## Documentation Standards

### Code Comments
```python
# Good: Explain why, not what
# Calculate orbital velocity using NASA's standard formula
velocity = math.sqrt(gravitational_constant * mass / radius)

# Avoid: Comments that state the obvious
# Set velocity to the result of the calculation
velocity = math.sqrt(gravitational_constant * mass / radius)
```

### Function Documentation
```python
def calculate_orbital_period(semi_major_axis: float, central_mass: float) -> float:
    """
    Calculate the orbital period using Kepler's third law.
    
    Args:
        semi_major_axis: The semi-major axis of the orbit in meters
        central_mass: The mass of the central body in kilograms
        
    Returns:
        The orbital period in seconds
        
    Raises:
        ValueError: If semi_major_axis or central_mass is negative
        
    Example:
        >>> period = calculate_orbital_period(150e9, 1.989e30)
        >>> print(f"Earth's orbital period: {period / (365.25 * 24 * 3600):.2f} years")
    """
```

## File Organization

### Directory Structure
```
src/
├── components/          # Reusable components
├── services/           # Business logic and API calls
├── utils/              # Utility functions
├── types/              # Type definitions
└── tests/              # Test files
```

### File Naming
- Use descriptive names
- Use kebab-case for files: `data-processor.py`
- Use PascalCase for classes: `DataProcessor.py`

## Testing Standards

### Test Structure
- Arrange, Act, Assert pattern
- One assertion per test (when possible)
- Descriptive test names

```python
def test_calculate_orbital_period_with_valid_inputs():
    # Arrange
    semi_major_axis = 150e9  # Earth's semi-major axis
    central_mass = 1.989e30  # Sun's mass
    
    # Act
    period = calculate_orbital_period(semi_major_axis, central_mass)
    
    # Assert
    expected_period = 365.25 * 24 * 3600  # One year in seconds
    assert abs(period - expected_period) < 1000  # Within 1000 seconds
```

### Test Coverage
- Maintain minimum 80% test coverage
- Test edge cases and error conditions
- Include integration tests for key workflows

## Git Standards

### Commit Messages
```
type(scope): brief description

Longer description if needed

- Bullet points for additional details
- Reference issue numbers: Closes #123
```

#### Types
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes
- `refactor`: Code refactoring
- `test`: Test additions or modifications
- `chore`: Maintenance tasks

#### Examples
```
feat(api): add NASA data fetching endpoint

- Implement connection to NASA Open Data API
- Add error handling for API failures
- Include rate limiting for API calls

Closes #45
```

### Branch Naming
- `feature/short-description`
- `bugfix/issue-number-description`
- `hotfix/critical-issue-description`

## Performance Standards

### General Guidelines
- Optimize for readability first, then performance
- Profile before optimizing
- Document performance-critical sections

### Specific Requirements
- API response time: < 2 seconds
- Page load time: < 3 seconds
- Memory usage: Monitor and document

## Security Standards

### General Security
- Never commit secrets or API keys
- Validate all user inputs
- Use HTTPS for all external communications
- Follow OWASP guidelines

### NASA Data Security
- Respect NASA API terms of service
- Implement proper data caching strategies
- Handle sensitive mission data appropriately

---
*Last updated: [Date]*