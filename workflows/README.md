# Workflow examples

## Meraki Authentication

- Authentication: "**Generic Credential Type**"
- Generic Auth Type: Most common is "**Header Auth**"
- Header Auth: Go at the bottom of the existing list and pick "**Create new credential**"
- Enter a **name**: Authorization
- **Value**: "Bearer your_api_key" - your_api_key is what you get from the API service (like for Meraki for example)

Then you can click top left to change the default name "Header Auth account" to something you like, in this example "Meraki Header Auth account".

## Catalyst SD-WAN Manager Authentication

Use Authentication with:

- Authentication: "**Generic Credential Type**"
- Generic Auth Type: "**Custom Auth**"
- Custom Auth: Go at the bottom of the existing list and pick "**Create new credential**"

and add JSON:

`{"headers": {"Content-Type":"application/x-www-form-urlencoded"}, "body" : {"j_username": "your-admin", "j_password":"your-password"}}`

Then "Send Header" with:

- Name: Content-Type
- Value: application/x-www-form-urlencoded

Then "Send Body":

Options: Ignore SSL Issues
