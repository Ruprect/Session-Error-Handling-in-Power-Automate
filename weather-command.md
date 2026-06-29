# Task: Two Open-Meteo Weather Flows (Success + Failure) with Error Handling

Build two Power Automate flows that retrieve current weather for Copenhagen from
Open-Meteo. Both must have error handling — one succeeds, one fails by design.

## Conventions to use

- **Use my error handling skill** to implement the error handling in both flows.
- **Use the flow template** for creating the flows (build both from the template,
  do not hand-roll the flow structure).

## Solution / ALM structure

- Create the two new flows in a **new solution** (e.g. `WeatherDemo`).
- The error handling must **reference the helper flows in the existing error
  handler solution** — call the existing helpers (child-flow invocation / the
  helper flows' run action). Do **not** duplicate the helper logic into the new
  solution.
- Declare the dependency on the error handler solution correctly so the new
  solution imports cleanly (helper flows referenced as existing components, not
  copied).

## Context

- Open-Meteo requires no API key, no auth, no headers. Plain HTTP GET returning
  JSON. Free for non-commercial use under CC BY 4.0 (attribution required).
- Base URL: `https://api.open-meteo.com/v1/forecast`
- Copenhagen: latitude `55.6761`, longitude `12.5683`.

## Flow 1 — succeeds (`Weather-Copenhagen-Success`)

1. Built from the flow template.
2. Trigger: manual (button).
3. HTTP GET to:
   `https://api.open-meteo.com/v1/forecast?latitude=55.6761&longitude=12.5683&current=temperature_2m,relative_humidity_2m,wind_speed_10m,weather_code&timezone=Europe/Copenhagen`
4. Parse JSON — the `current` object contains `time`, `temperature_2m`,
   `relative_humidity_2m`, `wind_speed_10m`, `weather_code`.
5. Compose the parsed temperature/humidity/wind into a readable output.
6. Error handling applied via my skill, calling the helper flows from the error
   handler solution. On a normal run the error path should not fire.

## Flow 2 — fails by design (`Weather-Copenhagen-Failure`)

1. Built from the flow template, identical structure to Flow 1, same error
   handling via the helper flows.
2. Use a deliberately wrong URL so the call fails — e.g.:
   `https://api.open-meteo.com/v1/forecastXXX?latitude=55.6761&longitude=12.5683&current=temperature_2m`
   (the `forecastXXX` path returns a 4xx, exercising the error path).
3. The error handler should fire, capture the status code and error body, and
   surface a clear "weather fetch failed" message via the helper flow.

## Deliverables

- Both flows in the new `WeatherDemo` solution, built from the template,
  referencing the helper flows in the error handler solution.
- Solution dependencies set so the helper flows are referenced, not duplicated.
- Verification notes:
  - Flow 1 completes green with weather values.
  - Flow 2 triggers the error handling and the helper flow handles the failure.

## Notes

- `weather_code` is a WMO code; optionally map to text.
- No secrets needed — Open-Meteo requires none.