---
title: "One year with NixOS"
date: 2024-02-21T14:15:29-08:00
draft: true
---
# The good

{{< highlight nixos >}}
nixpkgs.overlays = with inputs; [
    emacs-overlay.overlay 
];
{{< /highlight >}}

```nixos
sup
```
