# Salesforce CLI Configuration

This action allows you to configure Salesforce CLI using credential.

- # [Quick Start Prerequisites](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/quickstart_prereq.htm)
  - Confirm That You Have API Enabled Permission
  - [Create a Connected App](https://developer.salesforce.com/docs/atlas.en-us.api_iot.meta/api_iot/qs_auth_connected_app.htm)
  - Get a Consumer Key(`sfdc_client_id`) and a Consumer Secret(`sfdc_client_secret`)
  
- ## Inputs

  | Inputs              | Required  | Description |
  | :---                | :---:     | :---        |
  | sfdc_client_id      | ✅ | The consumer key of the connected app. To access the consumer key, from the App Manager, find the connected app and select View from the dropdown. Then click Manage Consumer Details. You're sometimes prompted to verify your identity before you can view the consumer key. |
  | sfdc_client_secret  | ✅ | The consumer secret of the connected app. To access the consumer secret, from the App Manager, find the connected app and select View from the dropdown. Then click Manage Consumer Details. You're sometimes prompted to verify your identity before you can view the consumer secret. |
  | username            | ✅ | The username of the user that the connected app is imitating. |
  | password            | ✅ | The password of the user that the connected app is imitating. |
  | login_url           | ✖️ | **Production** \| **Developer** \| **Devhub** -- `https://login.salesforce.com` `🟢 default` <br> **Sandbox** \| **Scratch** \| -- `https://test.salesforce.com` <br> **MyDomainName** `https://MyDomainName.my.salesforce.com` |

- ## Inputs suggestions
  Use following inputs with **secrets**, [Creating encrypted secrets for a repository](https://docs.github.com/en/actions/security-guides/encrypted-secrets#creating-encrypted-secrets-for-a-repository) | [Using encrypted secrets in a workflow](https://docs.github.com/en/actions/security-guides/encrypted-secrets#using-encrypted-secrets-in-a-workflow).
  - sfdc_client_id
  - sfdc_client_secret
  - password

- ## Example usage

```yaml
# SFDX configuration + source validation targeting user
name: SFDC CI CD with Salesforce CLI
on: [push]
jobs:
  sfdx_ci_cd:
    runs-on: ubuntu-latest
    name: SFDX usage with Git
    steps:
      # use GitHub Action to configure SFDX.
      - name: Salesforce CLI Setup
        uses: karamchandanid/sfdx-setup@v1
        id: sfdx_setup
        with:
          sfdc_client_id: ${{ secrets.SFDC_CLIENT_ID }}
          sfdc_client_secret: ${{ secrets.SFDC_CLIENT_SECRET }}
          user_name: ${{ secrets.SFDC_DEV_USERNAME }}
          password: ${{ secrets.SFDC_DEV_PASSWORD }}
          login_url: https://login.salesforce.com
      
      # Checkout the source code, need this to get repo, to pass "This directory does not contain a valid Salesforce DX project."
      - name: 'Checkout source code'
        uses: actions/checkout@v2
      
      # valdating source to org
      - name: 'Validating Source to org'
        run: |
          if [[ ${{ steps.sfdx_setup.outputs.sfdx_configured }} == 'true' ]]
          then
            # following usage fir Demo only, "SFDX version" | "Displaying Org details" | "Displaying User details"
            echo "SFDX version"
            echo "-------------------------"
            sfdx -v
            echo "Displaying Org details..."
            echo "-------------------------"
            sfdx force:org:display
            
            echo "Displaying User details..."
            echo "-------------------------"
            sfdx force:user:display

            # echo "Validating source to target org..."
            # echo "-------------------------"
            # no need to specify --targetusername
            sfdx force:source:deploy -p force-app/main/default -c
            
          else
            echo ${{ steps.sfdx_setup.outputs.error }}
            echo ${{ steps.sfdx_setup.outputs.error_description }}
          fi
