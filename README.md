## Machines à états finis - Chenillard à LEDs

### Description
Dans ce laboratoire, nous implémentons un chenillard à LEDs en utilisant une machine à états finis (FSM) sur la carte DE10-lite. Les LEDs s'allument de gauche à droite, puis effectuent le chemin inverse avec un délai de 500 ms.

#### Caractéristiques principales :
- **Horloge** : Fonctionne à 50 MHz.
- **Réinitialisation** : Utilisation d'un bouton pour réinitialiser la FSM.
- **Sorties LEDs** : Les LEDs sont mappées de LED<0> à LED<9>.
- **Nombre d'états** : `idle`, `ready`, `sl` (shift left), et `sr` (shift right).

### Code VHDL

#### Module Principal (`ledss.vhd`)
```vhdl
LIBRARY ieee;
USE ieee.std_logic_1164.ALL;
USE ieee.numeric_std.ALL;

ENTITY ledss IS
  GENERIC (
    N : NATURAL := 10
  );
  PORT (
    rst, clk, start : IN STD_LOGIC;
    dout : OUT STD_LOGIC_VECTOR(N - 1 DOWNTO 0)
  );
END ENTITY;

ARCHITECTURE fsm OF ledss IS
  TYPE states IS (idle, ready, sl, sr);
  SIGNAL cs, ns : states;
  SIGNAL currentled, nextled : unsigned(N - 1 DOWNTO 0);
BEGIN
  PROCESS (rst, clk)
  BEGIN
    IF (rst = '0') THEN
      cs <= idle;
      currentled <= (OTHERS => '0');
    ELSIF (clk'event AND clk = '1') THEN
      cs <= ns;
      currentled <= nextled;
    END IF;
  END PROCESS;

  PROCESS (cs, currentled)
  BEGIN
    CASE cs IS
      WHEN idle =>
        nextled <= (OTHERS => '0');
      WHEN ready =>
        nextled <= "0000000001";
      WHEN sl =>
        nextled <= shift_left(currentled, 1);
      WHEN sr =>
        nextled <= shift_right(currentled, 1);
    END CASE;
  END PROCESS;

  PROCESS (cs, start, currentled)
  BEGIN
    CASE cs IS
      WHEN idle =>
        IF (start = '0') THEN
          ns <= idle;
        ELSE
          ns <= ready;
        END IF;

      WHEN ready =>
        ns <= sl;

      WHEN sl =>
        IF (currentled < 256) THEN
          ns <= sl;
        ELSE
          ns <= sr;
        END IF;

      WHEN sr =>
        IF (currentled > 2) THEN
          ns <= sr;
        ELSE
          ns <= sl;
        END IF;

    END CASE;
  END PROCESS;

  dout <= STD_LOGIC_VECTOR(currentled);
END fsm;
```

#### Testbench (`ledss_tb.vhd`)
```vhdl
LIBRARY ieee;
USE ieee.std_logic_1164.ALL;

ENTITY ledss_tb IS
END ENTITY;

ARCHITECTURE tb OF ledss_tb IS
  COMPONENT ledss IS
    GENERIC (
      N : NATURAL := 10
    );
    PORT (
      rst, clk, start : IN STD_LOGIC;
      dout : OUT STD_LOGIC_VECTOR(N - 1 DOWNTO 0)
    );
  END COMPONENT;

  CONSTANT N : NATURAL := 10;
  SIGNAL rst_tb : STD_LOGIC;
  SIGNAL clk_tb : STD_LOGIC := '0';
  SIGNAL start_tb : STD_LOGIC := '1';
  SIGNAL dout_tb : STD_LOGIC_VECTOR(N - 1 DOWNTO 0);

  CONSTANT hp : TIME := 50 ns;
BEGIN
  DUT : ledss
  GENERIC MAP(10)
  PORT MAP(
    rst => rst_tb,
    clk => clk_tb,
    start => start_tb,
    dout => dout_tb
  );

  clk_tb <= NOT clk_tb AFTER hp;
  rst_tb <= '0', '1' AFTER 2 * hp;
END tb;
```

### Simulation avec ModelSim
1. Compilez les fichiers `ledss.vhd` et `ledss_tb.vhd`.
2. Lancez la simulation et observez le comportement des LEDs dans le chronogramme.

#### Chronogramme :
- Initialement, toutes les LEDs sont éteintes.
- Lors de l'activation de `start`, les LEDs s'allument de gauche à droite, puis de droite à gauche avec un délai d'environ 500 ms.

![Picture](https://github.com/user-attachments/assets/6d010c17-79a7-4278-aabe-92e3b070cd4c)


### Synthèse avec Quartus Prime Lite
1. Ouvrez Quartus Prime Lite.
2. Créez un nouveau projet et ajoutez `ledss.vhd` comme fichier source.
3. Configurez le mappage des broches pour la carte DE10-lite (voir manuel).
4. Générez la netlist.

#### Netlist générée
Ajoutez ici la netlist générée par Quartus Prime Lite.

  ![Capture d'écran 2024-12-14 154438](https://github.com/user-attachments/assets/1ef58dd6-d174-4062-ae21-f3a10f3c9c17)

### Résultats
Les LEDs se comportent comme prévu, offrant un chenillard bidirectionnel synchronisé avec l'horloge de la carte DE10-lite.
