---
title: FlexMeasures
description: Use FlexMeasures flexible energy asset in Home Assistant 
ha_category:
  - Car
ha_release: "2023.9"
ha_iot_class: Cloud Polling
ha_codeowners:
  - '@filcole'
ha_domain: flexmeasures
ha_platforms:
  - sensor
ha_integration_type: integration
ha_config_flow: true
ha_quality_scale: silver
---




The FlexMeasures integration offers an integration with [FlexMeasures](https://flexmeasures.io/) instances to schedule flexible energy assets.

FlexMeasures can be used to schedule batteries, EV's, heat storage and other assets. The integration offers:

- A sensor for flexible asset schedules
- A service to retrieve a schedule
- A service to post measurements to a FlexMeasures instance
- A service to change an S2 control type



To add this integration the following configuration input is needed:

```yaml
# Example configuration.yaml entry
flexmeasures:
  host: "FLEXMEASURES_INSTANCE"
  username: "USERNAME"
  password: "PASSWORD"
  power_sensor: "POWER_SENSOR_ID"
  consumption_price_sensor: "CONSUMPTION_PRICE_SENSOR_ID"
  production_price_sensor: "PRODUCTION_PRICE_SENSOR_ID"
  soc_sensor: "STATE_OF_CHARGE_SENSOR_ID"
  rm_discharge_sensor: "RESOURCE_MANAGER_DISCHARGE_SENSOR_ID"
  schedule_duration: "SCHEDULE_DURATION"
  soc_unit: "STATE_OF_CHARGE_UNIT"
  soc_min: "STATE_OF_CHARGE_MINIMUM"
  soc_max: "STATE_OF_CHARGE_MAXIMUM"
```

{% configuration %}
host:
  description: URI of the FlexMeasures instance, for instance (flexmeasures.io)
  required: true
  type: string
username:
  description: The username associated with the FlexMeasures instance account.
  required: true
  type: string
password:
  description: The password with the FlexMeasures instance account.
  required: true
  type: string
power_sensor:
  description: The power sensor that will be scheduled to draw or supply power.
  required: true
  type: integer
power_sensor:
  description: The power sensor that will be scheduled to draw or supply power.
  required: true
  type: integer
update_interval:
  description: The interval between updates if the climate control is off and the car is not charging. Set in any time unit (e.g.,  minutes, hours, days!). Providing a low interval will cause the service to refresh more frequently and can negatively impact your 12V battery. 
  required: false
  default: 1 hour
  type: time
update_interval_charging:
  description: The interval in minutes between updates if charging.
  required: false
  default: 15
  type: time
update_interval_climate:
  description: The interval in minutes between updates if climate control on.
  required: false
  default: 5
  type: time
{% endconfiguration %}

{% include integrations/config_flow.md %}

## Sensors

The integration adds the following sensors only if your utility provides forecasted usage/cost:

For electricity:

- Current bill electric usage to date
- Current bill electric cost to date
- Current bill electric forecasted usage (for the first few days of the bill this is 0)
- Current bill electric forecasted cost (for the first few days of the bill this is 0)
- Typical monthly electric usage
- Typical monthly electric cost

For gas:

- Current bill gas usage to date
- Current bill gas cost to date
- Current bill gas forecasted usage (for the first few days of the bill this is 0)
- Current bill gas forecasted cost (for the first few days of the bill this is 0)
- Typical monthly gas usage
- Typical monthly gas cost

Note the unit for gas is CCF (centum cubic feet). 1 CCF is one hundred cubic feet which is equivalent to 1 therm.

## Energy

Because utilities only release usage/cost data with a 48-hour delay, the integration inserts data into statistic objects.
You can find the statistics in {% my developer_statistics title="**Developer Tools** > **Statistics**"%} and search for "opower".
**This delay means that there will be no data in the energy dashboard for today and likely yesterday** (depending on time of day you are checking).

At the initial setup, the integration pulls historical monthly usage/cost since the account activation. If the utility provides more granular data, it pulls daily usage/cost for the past 3 years and hourly usage/cost for the past 2 months (note: typically, utilities provide only monthly or daily data for gas).
After the initial setup, the integration keeps pulling data (twice per day) for the past 30 days to allow for any corrections in the data from the utilities.

In the configuration of the energy dashboard (**{% my config_energy title="Settings > Dashboards > Energy" %}**):

For electricity:

1. Select **Add consumption** for the **Electricity grid**.
2. Select **Opower {utility name} elec {account number} consumption** for the **consumed energy**.
3. Select the radio button to **Use an entity tracking the total costs**.
4. Select **Opower {utility name} elec {account number} cost** for the **entity with the total costs**.

Your **Configure grid consumption** should now look like this:
![Screenshot configure grid consumption](/images/integrations/opower/configure_grid_consumption.png)

For gas:

1. Select **Add gas source** for the **Gas consumption**.
2. Select **Opower {utility name} gas {account number} consumption** for the **gas usage**.
3. Select the radio button to **Use an entity tracking the total costs**.
4. Select **Opower {utility name} gas {account number} cost** for the **entity with the total costs**.

Your **Configure gas consumption** should now look like this:
![Screenshot configure gas consumption](/images/integrations/opower/configure_gas_consumption.png)

With the above changes your (**{% my config_energy title="Settings > Dashboards > Energy" %}**) page should now look like this:

![Screenshot Energy Configuration](/images/integrations/opower/energy_config.png)
