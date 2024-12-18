## Player Color Change

- "E" tuşuna bastığımızda karakterimizin rengini (tüm clientler için) değiştiren kodu yazıyoruz.

```c#
using Fusion;
using UnityEngine;

public class PlayerColor : NetworkBehaviour
{
    public MeshRenderer playerBody;

    [Networked, OnChangedRender(nameof(RenkDegisti))] 
    public Color playerRenk { get; set; }

    private void Update()
    {
        if(HasStateAuthority)
        {
            if(Input.GetKeyDown(KeyCode.E))
            {
                playerRenk = new Color(Random.Range(0, 1f), Random.Range(0, 1f), Random.Range(0, 1f), 1f);
            }
        }
    }
    
    void RenkDegisti()
    {
        playerBody.material.color = playerRenk;
    }

}

```
