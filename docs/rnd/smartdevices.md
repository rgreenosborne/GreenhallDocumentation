# Smart Devices

## Sonoff T2US3C
Remake or Sonoff T1 3-gang. 

```
GPIO0 EFM8BB1 P1,3 Switch 1 input (Goes low when first touch button is pressed)
GPIO04 is connected to the small (soft) reset button on the front.
GPIO09 EFM8BB1 P1,4 - Switch 3 input (Goes low when third touch button is pressed)
GPIO10 EFM8BB1 P1,5 - Switch 2 input
GPIO13 is connected to status LED D3.
GPIO12 Relay 1
GPIO5 Relay 2
GPIO4 Relay 3
GPIO2 is connected on J3 pin 5 (LOG)
```

## Custom Rule for pulling high buttons.
event name: touchzero#switch
```
on touchzero#switch do
  if [touchzero#switch]=1
   gpio,12,1
  else
   gpio,12,0
  endif
 endon
```

