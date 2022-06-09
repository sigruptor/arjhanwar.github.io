## Permissions

#### Starting with IAM Policies
- Principals
- Policies
- Conditions
- Identity or Resource based, permission boundaries, "service control"


###### Strategies
More, Account, less policies
- Tree like collection of account for different organizations.
- Easy cost story - no tagging needed.
- No limit for API

Centrally manage principals and permissions
- Ideally: pushh/pull users as principals from some IdP (via SaML)
- Centrally manage and distribute roles
- Map users and groups to accounts and roles as "permission sets"

Customize beyond distributing permission sets
- AWS CloudFormation Stacks and StackSets

Write complex policies only for well-defined (or high stakes) scenarios.
Rely on review mechanisms instead of trying to regulate every interaction (or not regulate anything at all)
