# Build retention

Each [Buildkite plan](https://buildkite.com/pricing) has a maximum build retention period. Once builds reach the retention period, their data is removed from Buildkite.

The following diagram shows the lifecycle of build data by plan.

<%= image "build-retention-flow-chart.png", alt: "Simplified flow chart of the build retention process" %>

## Retention periods

<table width="100%">
  <thead>
    <tr>
      <th>Plan</th>
      <th>Retention period</th>
      <th>Supports build exports</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Developer/Free plan</td>
      <td>90 days</td>
      <td>-</td>
    </tr>
    <tr>
    <tr>
      <td>Open source plan</td>
      <td>1 year</td>
      <td>-</td>
    </tr>
    <tr>
      <td>Non-profits & charities plan</td>
      <td>1 year</td>
      <td>-</td>
    </tr>
      <td>Team plan</td>
      <td>1 year</td>
      <td>-</td>
    </tr>
    <tr>
      <td>Business plan</td>
      <td>1 year</td>
      <td>-</td>
    </tr>
    <tr>
      <td>Enterprise plan</td>
      <td>1 year</td>
      <td>Yes</td>
    </tr>
  </tbody>
</table>

## Exporting build data

> 📘 Enterprise feature
> Exporting build data is only available on an [Enterprise](https://buildkite.com/pricing) plan.

If you need to retain build data beyond the retention period in your [Buildkite plan](https://buildkite.com/pricing), you can have Buildkite export the data to a private Amazon S3 bucket or Google Cloud Storage (GCS) bucket. As build data is removed, Buildkite exports JSON representations of the builds to the bucket you provide. To learn more, see [Build exports](/docs/pipelines/build-exports).
