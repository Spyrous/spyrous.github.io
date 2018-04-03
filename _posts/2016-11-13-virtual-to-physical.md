---
layout: post
title: Test2
comments: true
categories: [Vivado, FPGA, VHDL, AXI]

---



Lanyon is an unassuming [Jekyll](http://jekyllrb.com) theme that places content first by tucking away navigation in a hidden drawer. It's based on [Poole](http://getpoole.com), the Jekyll butler.

<pre>
  <code class="VHDL">
library ieee;
use ieee.std_logic_1164.all;

entity RS_trigger is
    generic (T: Time := 0ns);
    port ( R, S  : in  std_logic;
           Q, nQ : out std_logic;
           reset, clock : in  std_logic );
end RS_trigger;

architecture behaviour of RS_trigger is
    signal QT: std_logic; -- Q(t)
begin
    process(clock, reset) is
        subtype RS is std_logic_vector (1 downto 0);
    begin
        if reset = '0' then
            QT <= '0';
        else
            if rising_edge(C) then
                if not (R'stable(T) and S'stable(T)) then
                    QT <= 'X';
                else
                    case RS'(R&S) is
                        when "01" => QT <= '1';
                        when "10" => QT <= '0';
                        when "11" => QT <= 'X';
                        when others => null;
                    end case;
                end if;
            end if;
        end if;
    end process;

    Q  <= QT;
    nQ <= not QT;
end architecture behaviour;
  </code>
</pre>
<pre>
  <code class="verilog">
`ifndef _alu
`define _alu

/*`include "rc_adder.v"*/

module alu(
        input       [3:0]   ctl,
        input       [31:0]  a, b,
        output reg  [31:0]  out,
        output              zero);

    wire [31:0] sub_ab;
    wire [31:0] add_ab;
    wire        oflow_add;
    wire        oflow_sub;
    wire        oflow;
    wire        slt;

    assign zero = (0 == out);

    assign sub_ab = a - b;
    assign add_ab = a + b;
    /*rc_adder #(.N(32)) add0(.a(a), .b(b), .s(add_ab));*/

    // overflow occurs (with 2s complement numbers) when
    // the operands have the same sign, but the sign of the result is
    // different.  The actual sign is the opposite of the result.
    // It is also dependent on whether addition or subtraction is performed.
    assign oflow_add = (a[31] == b[31] && add_ab[31] != a[31]) ? 1 : 0;
    assign oflow_sub = (a[31] == b[31] && sub_ab[31] != a[31]) ? 1 : 0;

    assign oflow = (ctl == 4'b0010) ? oflow_add : oflow_sub;

    // set if less than, 2s compliment 32-bit numbers
    assign slt = oflow_sub ? ~(a[31]) : a[31];

    always @(*) begin
        case (ctl)
            4'd2:  out <= add_ab;               /* add */
            4'd0:  out <= a & b;                /* and */
            4'd12: out <= ~(a | b);             /* nor */
            4'd1:  out <= a | b;                /* or */
            4'd7:  out <= {{31{1'b0}}, slt};    /* slt */
            4'd6:  out <= sub_ab;               /* sub */
            4'd13: out <= a ^ b;                /* xor */
            default: out <= 0;
        endcase
    end

endmodule

`endif
  </code>
</pre>

### Built on Poole

Poole is the Jekyll Butler, serving as an upstanding and effective foundation for Jekyll themes by [@mdo](https://twitter.com/mdo). Poole, and every theme built on it (like Lanyon here) includes the following:

* Complete Jekyll setup included (layouts, config, [404](/404), [RSS feed](/atom.xml), posts, and [example page](/about))
* Mobile friendly design and development
* Easily scalable text and component sizing with `/etc/scripts/run_command.sh` units in the CSS
* Support for a wide gamut of HTML elements
* Related posts (time-based, because Jekyll) below each post
* Syntax highlighting, courtesy Pygments (the Python-based code snippet highlighter)

### Lanyon features

In addition to the features of Poole, Lanyon adds the following:

* Toggleable sliding sidebar (built with only CSS) via **☰** link in top corner
* Sidebar includes support for textual modules and a dynamically generated navigation with active link support
* Two orientations for content and sidebar, default (left sidebar) and [reverse](https://github.com/poole/lanyon#reverse-layout) (right sidebar), available via `<body>` classes
* [Eight optional color schemes](https://github.com/poole/lanyon#themes), available via `<body>` classes

[Head to the readme](https://github.com/poole/lanyon#readme) to learn more.

### Browser support

Lanyon is by preference a forward-thinking project. In addition to the latest versions of Chrome, Safari (mobile and desktop), and Firefox, it is only compatible with Internet Explorer 9 and above.

### Download

Lanyon is developed on and hosted with GitHub. Head to the <a href="https://github.com/poole/lanyon">GitHub repository</a> for downloads, bug reports, and features requests.

Thanks!

