Tips:
```
   env:
     - name: BAR
       valueFrom:
         configMapKeyRef:
           name: yourconfigmap
           key: BAR
```
