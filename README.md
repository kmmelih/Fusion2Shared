## Sağlık Sistemi 

- Oyundayken bir başka oyuncuya ateş ettğimiz zaman hasar veren kodu yazalım.
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
- Üzerine ateş ettiğimizi kontrol eden Raycasting sistemini yazalım.

```c#
using Fusion;
using UnityEngine;

public class RayAttack : NetworkBehaviour
{
    public PlayerMovement playerMovement;
    public float damage = 10f;
    public float range = 100f;
    

    private void Update()
    {
        if(!HasStateAuthority) {  return; }

        Ray ray = new Ray(Camera.main.transform.position, Camera.main.transform.forward);

        if (Input.GetKeyDown(KeyCode.Mouse0))
        {
            Debug.DrawRay(ray.origin,ray.direction * range,Color.red,5f);
            if(Runner.GetPhysicsScene().Raycast(ray.origin,ray.direction * range, out var hit))
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
