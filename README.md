# (Photon) Fusion 2 Shared Ders Notları

- Projemizi oluşturduktan sonra Hierarchy üzerine sağ tıklayıp **Fusion > Scene > Setup Networkin in the Scene** seçeneğini seçiyoruz.
- Prototype Network Start ve Prototype Runner adında 2 adet objeyi bizim için oluşturacak.
- Player Spawner kodumuzu yazıp Prototype Runner objesine ekleyeceğiz.

## Player Spawner

```c#
using Fusion;
using UnityEngine;

public class PlayerSpawner : SimulationBehaviour, IPlayerJoined
{
    [SerializeField] private GameObject playerCharacter;
    public void PlayerJoined(PlayerRef player)
    {
        Runner.Spawn(playerCharacter, new Vector3(0, 1f, 0), Quaternion.identity);
    }
}
```
- Oyunumuzu Shared modunda çalıştırdığımızda sunucunun bizim belirlediğimiz Prefabı Spawn ettiğini görebiliriz.

