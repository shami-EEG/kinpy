# Introduction #
The problem with solving the equations in classical Chemical kinetics isn't that of computational power, but the amount of time it takes to manually write code for each particular set of reactions which need to be solved numerically - just try to write the code for a reaction system composed of 10 elementary reactions, if you don't believe me!
Scalability is something to look for if you're trying to solve anything more than 3-4 equations.

Kinpy does not address this problem completely, as do many standalone programs with GUI interfaces; it is meant for those who'd want to know everything that goes on underneath, without much learning. Hence this is mainly meant for the Python-Scipy-Matplotlib community.

Kinpy is a source code generator for solving Chemical Kinetic equations in Scipy using scipy.integrate.odeint. It can just as easily be adopted to any other integrator.
To do this, Kinpy parses a set of Chemical equations stored in a file - aptly a file with a ".k" identifier.

# Usage #
Say we want to explicitly solve the Michaelis Menten mechanism proposed for Enzyme reactions.

The elementary reactions are:
```
E + S <-> ES
ES -> E + P
```

To generate the code for solving for the concentrations of the chemical species involved,
put this in a file and call it "mm.k".
```
#Michaelis Menten Kinetics
E + S <-> ES
ES <-> E + P
```

That's it! Now we can generate code by running kinpy:
```
$ {path-to-kinpy}/kinpy mm.k
```

This generates a file called mm.py
```
#!/usr/bin/python

from scipy import *
import scipy.integrate as itg

'''
## Reaction ##

#Michaelis-Menten enzyme kinetics.

E + S <-> ES
ES <-> E + P

## Mapping ##

E	0	 -1*v_0(y[1], y[0], y[2]) +1*v_1(y[3], y[0], y[2])
S	1	 -1*v_0(y[1], y[0], y[2])
ES	2	 +1*v_0(y[1], y[0], y[2]) -1*v_1(y[3], y[0], y[2])
P	3	 +1*v_1(y[3], y[0], y[2])
'''

dy = lambda y, t: array([\
 -1*v_0(y[1], y[0], y[2]) +1*v_1(y[3], y[0], y[2]),\
 -1*v_0(y[1], y[0], y[2]),\
 +1*v_0(y[1], y[0], y[2]) -1*v_1(y[3], y[0], y[2]),\
 +1*v_1(y[3], y[0], y[2])\
])

#Initial concentrations:
y0 = array([\
#E
0.0,\
#S
0.0,\
#ES
0.0,\
#P
0.0,\
])

#E + S <-> ES
v_0 = lambda S, E, ES : k0 * E**1 * S**1 - k0r * ES**1
k0 = 1.0
k0r = 1.0

#ES <-> E + P
v_1 = lambda P, E, ES : k1 * ES**1 - k1r * E**1 * P**1
k1 = 1.0
k1r = 1.0

t = arange(0, 10, 0.01)
Y = itg.odeint(dy, y0, t)
```

After this, just edit the file to change the initial conditions and the kinetic constants. The individual reaction rates are given by the functions v_{reaction number}. The code should be pretty self-explanatory._

Kinpy assumes that the individual reactions are elementary, if not you can just change the function v_{reaction number} to suit your requirements._

You can also generate code for elementary reactions of different orders.
```
#Second Order, second_order.k
2*A <-> B
```

Make sure that the only character separating the stoichiometric co-efficient and the species name is the asterisk - no spaces.

You can find all these examples in the directory in the tests/ folder in the source code tree.


