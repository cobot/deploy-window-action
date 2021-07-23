# Deploy Window

Stop GitHub Actions deployments outside of a given time window.

If you are doing continious deployment using GitHub Actions, it might be a good
idea to not deploy on weekends or during the night.

## Usage

In your GitHub workflow file:

```yml
on: [push] # force deploy only works on the latest commit of a push event

jobs:
  should_deploy:
    outputs:
      result: ${{ steps.decide.outputs.deploy_or_not }}
    if: github.ref == 'refs/heads/main' # free idea: only deploy from main branch
    runs-on: ubuntu-latest
    steps:
      - id: decide
        uses: cobot/deploy-window-action@v1
        with: # all optional, see action.yml for default values
          earliest-hour: "9"
          latest-hour: "17"
          latest-weekday: "5"
          time-zone: UTC
          force-deploy-phrase: force deploy
  deploy:
    needs:
      - should_deploy
    if: needs.should_deploy.outputs.result == 'yes'
    steps:
      - uses: actions/checkout@v2
      - run: deploy.sh # run your deploy script
        shell: bash
  notify_skipped_deployment:
    runs-on: ubuntu-latest
    needs:
      - should_deploy
    if: needs.should_deploy.outputs.result == 'no'
    steps:
      - name: Slack notification
        env:
          SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK_URL }}
        uses: Ilshidur/action-slack@689ad44a9c9092315abd286d0e3a9a74d31ab78a
        with:
          args: ":warning: Deployment skipped. Now is not a good time. :warning:\nCommit: https://github.com/${{github.repository}}/commit/${{github.sha}}\nRe-run here: https://github.com/${{github.repository}}/actions/runs/${{github.run_id}}"
```
