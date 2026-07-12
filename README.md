# PCB Coil Designer

A single-file browser tool for designing PCB-based Helmholtz coil pairs and single-coil PCB magnetorquers. It lets you vary the board geometry, copper stackup, drive conditions, and target field, then recalculates magnetic field strength, field uniformity, resistance, voltage drop, current density, power, and magnetic dipole moment.

The app is intended for quick design exploration of ground-test magnetic-field rigs, such as ACS or magnetometer calibration fixtures.

## Quick Start

Open `pcb_helmholtz_designer.html` in a modern desktop browser.

No build step, package install, or server is required. The app is plain HTML, CSS, and JavaScript.

## Main Features

- Helmholtz pair and single-coil magnetorquer modes.
- Circular or square PCB coil geometry.
- PCB outer size up to 2000 mm.
- Trace width up to 250 mm.
- Manual, ideal Helmholtz, and max-stability coil separation modes.
- Copper layer count from 1 to 16 active winding layers per PCB.
- PCB stack count from 1 to 10 boards per side or per magnetorquer coil.
- Copper weight selector: 1, 2, 3, or 4 oz/ft^2.
- Drive modes for automatic safe current, fixed supply voltage, or manual current.
- Target magnetic field input for optimization.
- Uniformity cube side input for checking field variation over a centered volume.
- Lowest-power coil optimizer using multi-parameter gradient descent.
- 2D winding preview, 1D uniformity plots, and 3D uniformity surface view.
- IPC-2221-based current estimate with derating for low self-heating.

## Basic Workflow

1. Choose Helmholtz pair or magnetorquer mode.
2. Choose the coil shape: circular or square.
3. Set the PCB outer size, winding width, trace width, and clearance.
4. Set the number of copper layers, PCB stack count, and copper weight.
5. For a Helmholtz pair, choose a coil separation mode.
6. Choose the drive mode:
   - `Auto safe current`: uses the derated IPC-2221 current estimate.
   - `Use drive voltage`: fixes the supply voltage and calculates current from `I = V / R`.
   - `Manual current`: lets you directly enter the drive current.
7. Set the test volume and tolerance band to inspect field uniformity.
8. Use the outputs and plots to evaluate the design. Magnetorquer mode also reports magnetic dipole moment in A m^2.

## Optimizer

The optimizer searches over multiple design parameters:

- PCB outer size
- winding width
- trace width
- manual coil separation (Helmholtz mode only)

It uses the selected design mode, shape, copper layers, copper weight, board stack count, clearance, supply voltage, target magnetic field, and uniformity cube side. Coil separation is optimized only for a Helmholtz pair.

The drive mode changes how voltage is handled during optimization:

- `Use drive voltage`: fixes the supply voltage and searches for a geometry where direct-supply current, `I = V / R`, hits the target field.
- `Auto safe current` or `Manual current`: treats the supply voltage as the maximum available voltage and uses only the current required for the target field.

The optimizer is configured to prefer the lowest-power solution that satisfies:

- target magnetic field
- uniformity tolerance
- thermal current limit
- voltage constraint from the selected drive mode

After an optimization run, the app applies the result to the controls. A Helmholtz result switches to manual separation; separation does not apply to a magnetorquer. The app keeps voltage drive mode when the supply voltage is fixed.

## Supply Voltage vs Coil Voltage Drop

In `Use drive voltage` mode, the supply voltage is fixed. The app calculates direct-supply current from:

```text
I = Vsupply / Rcoil
```

and the optimizer searches for a coil geometry that hits the target magnetic field with that fixed supply.

In `Auto safe current` and `Manual current` modes, the supply voltage is treated as the maximum available voltage. The coil may only need part of that voltage if a current-regulated driver is used.

For example, with a 12 V supply:

- `Use drive voltage`: the optimizer assumes the coil is driven from 12 V, so current is `12 V / Rcoil`.
- `Manual current`: the optimizer can use less than 12 V if the requested current only drops 1 V or 2 V across the coil.

## Electrical Model

The app estimates:

- copper resistance from trace length, trace width, copper thickness, layer count, and board stack count
- coil voltage drop from `V = I * R`
- power dissipation from `P = I^2 * R`
- current density from copper cross-section
- IPC-2221 current limit using the worst simulated layer

Copper weight is applied equally to every active simulated winding layer.

## Magnetic Field Model

The magnetic field is calculated from the actual winding pattern using a Biot-Savart segment model. The app evaluates the center field and compares field magnitude at points inside a centered cube to estimate uniformity.

The 3D surface view shows percentage deviation from the center field over the selected test plane.

## Important Assumptions

- Helmholtz mode uses two symmetric coil sides.
- Magnetorquer mode uses one PCB stack centered on the field origin.
- Each PCB in a stack is treated as 1.6 mm thick.
- Winding layers on the same PCB are treated as geometrically coincident for field contribution.
- Copper pour/weight affects resistance, current density, IPC current sizing, and optimizer power, but not the geometric trace path.
- The IPC-2221 value is an estimate, not a thermal simulation.
- The model does not include vias, connectors, solder joints, copper temperature rise feedback, driver losses, magnetic materials, or enclosure effects.

## Useful Limits

- PCB outer size: 50 to 2000 mm
- Winding width: 5 to 390 mm
- Trace width: 0.1 to 250 mm
- Clearance: 0.1 to 6 mm
- Coil separation: 10 to 2500 mm
- Copper weight: 1 to 4 oz/ft^2
- Supply voltage: 0.1 to 1000 V
- Manual current: 0.01 to 200 A
- Target field: 0.01 to 100000 uT
- Uniformity cube side: 10 to 600 mm

## Files

- `pcb_helmholtz_designer.html`: the complete app.
- `README.md`: project documentation.

## Notes

Use this as an engineering exploration tool. Before building hardware, verify the final design with your PCB fabricator's stackup rules, copper-current guidance, thermal constraints, driver limits, and mechanical constraints.
