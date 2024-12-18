## Sağlık Sistemi 

- Oyundayken bir başka oyuncunun üzerine tıkladığımız zaman sağlığını azatlan sistemi yazıyoruz.
- Sunucu tarafında kontrol edilen sağlık sistemimizi yazalım.

```c#
using Fusion;
using UnityEngine;

public class Health : NetworkBehaviour
{
    [Networked, OnChangedRender(nameof(SaglikDegisti))]
    public float saglik { get; set; } = 100f;

    private void SaglikDegisti()
    {
        Debug.Log("Sağlık: "+saglik);
    }

    [Rpc(RpcSources.All,RpcTargets.StateAuthority)]
    public void HasarVerRpc(float d)
    {
        saglik -= d;
    }
}
```
- Üzerine tıkladığımızı kontrol eden Raycasting sistemini yazalım.

```c#
using Fusion;
using UnityEngine;

public class RayAttack : NetworkBehaviour
{
    public PlayerMovement playerMovement;
    public float damage = 10f;

    private void Update()
    {
        if(!HasStateAuthority) {  return; }

        Ray ray = playerMovement.camera.ScreenPointToRay(Input.mousePosition);
        ray.origin += playerMovement.transform.forward;

        if(Input.GetKeyDown(KeyCode.Mouse0))
        {
            Debug.DrawRay(ray.origin,ray.direction,Color.red,5f);
            if(Runner.GetPhysicsScene().Raycast(ray.origin,ray.direction, out var hit))
            {
                if(hit.transform.TryGetComponent<Health>(out var healt))
                {
                    healt.HasarVerRpc(damage);
                }
            }
        }
    }
}
```

- Bu yazdığımız kodlar sayesinde üzerine tıklanan oyunuya hasar verebiliriz.
