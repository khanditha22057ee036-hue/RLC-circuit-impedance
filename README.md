"""
RLC Circuit Impedance Calculator
--------------------------------
A menu-driven program for engineering students to analyse series and
parallel RLC circuits connected to an AC supply.

Key relations:
    Inductive reactance    XL = 2 * pi * f * L                (ohm)
    Capacitive reactance   XC = 1 / (2 * pi * f * C)          (ohm)

Series RLC circuit:
    Impedance              Z = R + j(XL - XC)
    |Z| = sqrt(R^2 + (XL - XC)^2)
    Phase angle            phi = arctan((XL - XC) / R)
    Current                I = V / |Z|
    XL > XC -> inductive (current lags)    XL < XC -> capacitive (current leads)
    XL = XC -> resonance, Z = R (minimum impedance)

Parallel RLC circuit (ideal branches):
    Admittance             Y = 1/R + 1/(jXL) + 1/(-jXC)
    Impedance              Z = 1 / Y
    At resonance Z = R (maximum impedance)

Resonance:
    Resonant frequency     f0 = 1 / (2 * pi * sqrt(L * C))
    Series quality factor  Q = (1 / R) * sqrt(L / C)
    Parallel quality factor Q = R * sqrt(C / L)
    Bandwidth              BW = f0 / Q
"""

import cmath
import math


# ---------------------------------------------------------------- input helpers
def get_positive_float(prompt):
    """Keep asking until the user enters a valid positive number."""
    while True:
        try:
            value = float(input(prompt))
            if value <= 0:
                print("  Please enter a value greater than zero.")
                continue
            return value
        except ValueError:
            print("  Invalid input. Please enter a number.")


def get_circuit_values():
    """Ask for R, L, C (converted to base units), frequency and voltage."""
    r = get_positive_float("Resistance R (ohm): ")
    l = get_positive_float("Inductance L (mH): ") / 1e3
    c = get_positive_float("Capacitance C (microfarad): ") / 1e6
    f = get_positive_float("Frequency f (Hz): ")
    v = get_positive_float("Supply voltage V (V, rms): ")
    return r, l, c, f, v


# ------------------------------------------------------------------ core maths
def inductive_reactance(f, l):
    """XL = 2 * pi * f * L"""
    return 2 * math.pi * f * l


def capacitive_reactance(f, c):
    """XC = 1 / (2 * pi * f * C)"""
    return 1 / (2 * math.pi * f * c)


def series_impedance(r, xl, xc):
    """Series RLC: Z = R + j(XL - XC) (complex number)."""
    return complex(r, xl - xc)


def parallel_impedance(r, xl, xc):
    """Parallel RLC: Z = 1 / (1/R + 1/(jXL) + 1/(-jXC))."""
    y = 1 / r + 1 / (1j * xl) + 1 / (-1j * xc)
    return 1 / y


def resonant_frequency(l, c):
    """f0 = 1 / (2 * pi * sqrt(L * C))"""
    return 1 / (2 * math.pi * math.sqrt(l * c))


def q_series(r, l, c):
    """Q = (1/R) * sqrt(L/C)"""
    return math.sqrt(l / c) / r


def q_parallel(r, l, c):
    """Q = R * sqrt(C/L)"""
    return r * math.sqrt(c / l)


# ------------------------------------------------------------------- display
def circuit_nature(xl, xc):
    """Describe the circuit behaviour at the given frequency."""
    if math.isclose(xl, xc, rel_tol=1e-3):
        return "At resonance (purely resistive)"
    if xl > xc:
        return "Inductive (current lags voltage)"
    return "Capacitive (current leads voltage)"


def show_impedance(z):
    """Print impedance in rectangular and polar form."""
    magnitude, angle = cmath.polar(z)
    print(f"  Impedance (rectangular) Z = {z.real:.3f} {'+' if z.imag >= 0 else '-'} j{abs(z.imag):.3f} ohm")
    print(f"  Impedance (polar)       Z = {magnitude:.3f} < {math.degrees(angle):.2f} deg ohm")
    print(f"  Power factor            = {math.cos(angle):.4f}")


def series_rlc():
    r, l, c, f, v = get_circuit_values()
    xl, xc = inductive_reactance(f, l), capacitive_reactance(f, c)
    z = series_impedance(r, xl, xc)
    i = v / abs(z)

    print("\n  ----- Series RLC Results -----")
    print(f"  Inductive reactance  XL = {xl:.3f} ohm")
    print(f"  Capacitive reactance XC = {xc:.3f} ohm")
    print(f"  Net reactance        X  = {xl - xc:.3f} ohm")
    show_impedance(z)
    print(f"  Nature                  = {circuit_nature(xl, xc)}")
    print(f"  Current I               = {i:.4f} A")
    print(f"  Voltage across R  (VR)  = {i * r:.3f} V")
    print(f"  Voltage across L  (VL)  = {i * xl:.3f} V")
    print(f"  Voltage across C  (VC)  = {i * xc:.3f} V")
    print(f"  Active power P          = {i ** 2 * r:.3f} W")


def parallel_rlc():
    r, l, c, f, v = get_circuit_values()
    xl, xc = inductive_reactance(f, l), capacitive_reactance(f, c)
    z = parallel_impedance(r, xl, xc)
    ir, il, ic = v / r, v / xl, v / xc
    i_total = math.sqrt(ir ** 2 + (ic - il) ** 2)

    print("\n  ----- Parallel RLC Results -----")
    print(f"  Inductive reactance  XL = {xl:.3f} ohm")
    print(f"  Capacitive reactance XC = {xc:.3f} ohm")
    show_impedance(z)
    nature = "Inductive (current lags)" if il > ic else "Capacitive (current leads)"
    if math.isclose(il, ic, rel_tol=1e-3):
        nature = "At resonance (purely resistive)"
    print(f"  Nature                  = {nature}")
    print(f"  Resistor current  IR    = {ir:.4f} A")
    print(f"  Inductor current  IL    = {il:.4f} A")
    print(f"  Capacitor current IC    = {ic:.4f} A")
    print(f"  Total current     I     = {i_total:.4f} A")


def resonance():
    r = get_positive_float("Resistance R (ohm): ")
    l = get_positive_float("Inductance L (mH): ") / 1e3
    c = get_positive_float("Capacitance C (microfarad): ") / 1e6

    f0 = resonant_frequency(l, c)
    qs, qp = q_series(r, l, c), q_parallel(r, l, c)

    print("\n  ----- Resonance Results -----")
    print(f"  Resonant frequency f0   = {f0:.2f} Hz")
    print(f"  Series Q factor         = {qs:.3f}")
    print(f"  Series bandwidth        = {f0 / qs:.2f} Hz")
    print(f"  Parallel Q factor       = {qp:.3f}")
    print(f"  Parallel bandwidth      = {f0 / qp:.2f} Hz")
    print(f"  Series impedance at f0  = {r:.3f} ohm (minimum)")
    print(f"  Parallel impedance at f0 = {r:.3f} ohm (maximum)")


def frequency_sweep():
    r = get_positive_float("Resistance R (ohm): ")
    l = get_positive_float("Inductance L (mH): ") / 1e3
    c = get_positive_float("Capacitance C (microfarad): ") / 1e6
    f0 = resonant_frequency(l, c)

    print(f"\n  Resonant frequency f0 = {f0:.2f} Hz")
    print(f"\n  {'f (Hz)':>10}{'XL':>10}{'XC':>10}{'|Z| series':>12}{'Angle (deg)':>13}")
    for factor in (0.25, 0.5, 0.8, 1.0, 1.25, 2.0, 4.0):
        f = f0 * factor
        xl, xc = inductive_reactance(f, l), capacitive_reactance(f, c)
        z = series_impedance(r, xl, xc)
        print(f"  {f:>10.1f}{xl:>10.2f}{xc:>10.2f}{abs(z):>12.2f}{math.degrees(cmath.phase(z)):>13.2f}")


def menu():
    print("\n" + "=" * 52)
    print("        RLC CIRCUIT IMPEDANCE CALCULATOR")
    print("=" * 52)
    print(" 1. Series RLC circuit")
    print(" 2. Parallel RLC circuit")
    print(" 3. Resonance (f0, Q factor, bandwidth)")
    print(" 4. Impedance vs frequency table (series RLC)")
    print(" 0. Exit")
    print("-" * 52)


def main():
    while True:
        menu()
        choice = input("Enter your choice: ").strip()

        if choice == "1":
            series_rlc()
        elif choice == "2":
            parallel_rlc()
        elif choice == "3":
            resonance()
        elif choice == "4":
            frequency_sweep()
        elif choice == "0":
            print("\nThank you for using the calculator. Goodbye!")
            break
        else:
            print("  Invalid choice. Please select from the menu.")


if __name__ == "__main__":
    main()
