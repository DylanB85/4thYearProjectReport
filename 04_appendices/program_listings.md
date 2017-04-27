
#Program Listings

##List of programs Included

* Buttons.cs

* CountDownTimer.cs

* DestroyPlayer.cs

* EnemyNav2.cs

* EnemyNav3.cs

* GameManager.cs

* PlayerDisplay.cs

* PlayerManager.cs

* spawnBlock.cs

```
using UnityEngine;
using System.Collections;
using UnityEngine.SceneManagement;

public class Buttons : MonoBehaviour
{
    public void MENU_ACTION_LoadDemo_GamePlaying() // method to call inside unity
    {
        SceneManager.LoadScene("Demo");  // load the scene inside the brackets
    }
}
```

```
using UnityEngine;
using System.Collections;

public class CountdownTimer : MonoBehaviour
{
    private float countdownTimerStartTime;
    private int countdownTimerDuration;

    public int GetTotalSeconds()
    {
        return countdownTimerDuration;
    }

    public void ResetTimer(int seconds)
    {
        countdownTimerStartTime = Time.time;
        countdownTimerDuration = seconds;
    }

    public int GetSecondsRemaining()
    {
        int elapsedSeconds = GetElapsedSeconds();
        int secondsLeft = (countdownTimerDuration - elapsedSeconds);
        return secondsLeft;
    }

    public int GetElapsedSeconds()
    {
        int elapsedSeconds = (int)(Time.time - countdownTimerStartTime);
        return elapsedSeconds;
    }

    public float GetProportionTimeRemaining()
    {
        float proportionLeft = (float)GetSecondsRemaining() / (float)GetTotalSeconds();
        return proportionLeft;
    }
}

```

```
using UnityEngine;
using System.Collections;

public class DestroyPlayer : MonoBehaviour
{
   
    void OnTriggerEnter (Collider hit)
    {
        if (hit.CompareTag("Player"))
        {
            PlayerManager playerManager = hit.gameObject.GetComponent<PlayerManager>();
            if (!playerManager.isInvincible)
            {
                Destroy(hit.gameObject);
            }
        }
    }
}
```

```
using UnityEngine;
using System.Collections;

public class EnemyNav2 : MonoBehaviour
{
    public float speed = 6.0F;
    public float gravity = 20.0F;
    private Vector3 moveDirection = Vector3.zero;
    void Update()
    {
        CharacterController controller = GetComponent<CharacterController>();
        if (controller.isGrounded)
        {
            moveDirection = new Vector3(Input.GetAxis("Horizontal1"), 0, Input.GetAxis("Vertical1"));
            moveDirection = transform.TransformDirection(moveDirection);
            moveDirection *= speed;

        }
        moveDirection.y -= gravity * Time.deltaTime;
        controller.Move(moveDirection * Time.deltaTime);
    }
}
```

```
using UnityEngine;
using System.Collections;

public class EnemyNav3 : MonoBehaviour
{
    public float speed = 6.0F;
    public float gravity = 20.0F;
    private Vector3 moveDirection = Vector3.zero;
    void Update()
    {
        CharacterController controller = GetComponent<CharacterController>();
        if (controller.isGrounded)
        {
            moveDirection = new Vector3(Input.GetAxis("Horizontal2"), 0, Input.GetAxis("Vertical2"));
            moveDirection = transform.TransformDirection(moveDirection);
            moveDirection *= speed;

        }
        moveDirection.y -= gravity * Time.deltaTime;
        controller.Move(moveDirection * Time.deltaTime);
    }
}
```

```
using UnityEngine;
using System.Collections;
using UnityEngine.UI;
using UnityEngine.SceneManagement;

public class GameManager : MonoBehaviour
{
    public Text Time_Remaining;
    private CountdownTimer countDownTimer; //reference to countdown timer
    private int totalSeconds = 60;

    void Start()
    {
        countDownTimer = GetComponent<CountdownTimer>();//
        countDownTimer.ResetTimer(totalSeconds);//
    }

    void Update()
    {
        string msg = "" + countDownTimer.GetSecondsRemaining();
        Time_Remaining.text = msg;
        checkGameOver();
    }

    private void checkGameOver()
    {
        if(countDownTimer.GetSecondsRemaining()<0)
        {
            SceneManager.LoadScene("scene1_Over");
        }
    }
}
```

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class PlayerDisplay : MonoBehaviour {
    public Text textBlue;
    public Text textRed;

    private PlayerManager player;

    private void Awake()
    {
        player = GetComponent<PlayerManager>();
    }

    public void UpdateBlue()
    {
        textBlue.text = "" + player.GetBlueTotal();
    }

    public void UpdateRed()
    {
        textRed.text = "No" + player.GetRedTotal();
    }

}

```

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class PlayerManager : MonoBehaviour
{

    private int blueTotal = 0;
    private int timeBetweenBlueSpawns = 10;
    public Text redNo;
    public Text redYes;
    private int timeBetweenRedSpawns = 10;

    public float freezeTime = 3f;
    public float invincibleTime = 10f;
    public bool isInvincible = false;

    public spawnBlock blockSpawner;
    public GameObject inventoryPanel;
    public GameObject[] inventoryIcons;
    private PlayerDisplay playerDisplay;

    public GameObject weapon;
    public GameObject bullet;
    public bool canFire = false;
    public float Bullet_Foward_Force;

    public void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("Blue"))
        {
            Destroy(other.gameObject);
            blockSpawner.SpawnOneBlockAfterDelay(timeBetweenBlueSpawns);
            blueTotal++;
            playerDisplay.UpdateBlue();
            Update();
            print("Blue Total Equals" + blueTotal);
        }

        if (other.CompareTag("Red"))
        {
            gameObject.tag = "Invincible";
            gameObject.layer = 13;
            Destroy(other.gameObject);
            blockSpawner.SpawnOneBlockAfterDelay(timeBetweenRedSpawns);
            playerDisplay.UpdateRed();
            SetInvincible();
        }

        if(other.CompareTag("Projectile"))
        {
            SetFreeze();
            DestroyObject(other.gameObject);
        }
    }

    public void SetInvincible()
    {
        isInvincible = true;
        CancelInvoke("SetDamagable");
        Invoke("SetDamagable", invincibleTime);
        redNo.text = " ";
        redYes.text = "Yes";
    }

    public void SetFreeze()
    {
        CancelInvoke("Unfreeze");
        this.GetComponent<CharacterController>().enabled = false;
        Invoke("Unfreeze", freezeTime);
    }

    void SetDamagable()
    {
        isInvincible = false;
        gameObject.tag = "Player";
        gameObject.layer = 12;
        redYes.text = "No";
    }

    void Unfreeze()
    {
        this.GetComponent<CharacterController>().enabled = true;
    }

    public int GetBlueTotal()
    {
        return blueTotal;
    }
    
    public string GetRedTotal()
    {
        return redYes.text;
    }

    public void Start()
    {
        playerDisplay = GetComponent<PlayerDisplay>();
        playerDisplay.UpdateBlue();
        playerDisplay.UpdateRed();
    }


    public void Update()
    {
        if (blueTotal > 0)
            canFire = true;
        if (blueTotal <= 0)
            canFire = false;

        if (Input.GetKeyDown("tab") && canFire == true)
            IsFiring();
        playerDisplay.UpdateBlue();
    }

    public void IsFiring()
    {
        if (blueTotal > 0 && canFire == true)
           blueTotal--;
        GameObject Temporary_Bullet_Handler;

        Temporary_Bullet_Handler=Instantiate(bullet, weapon.transform.position, weapon.transform.rotation);
        Temporary_Bullet_Handler.transform.Rotate(Vector3.left * 90);

        Rigidbody Temporary_RigidBody;
        Temporary_RigidBody = Temporary_Bullet_Handler.GetComponent<Rigidbody>();
        Temporary_RigidBody.AddForce(transform.forward * Bullet_Foward_Force);

        if(blueTotal ==0)
        {
            canFire = false;
        }

    }
}

```

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class spawnBlock : MonoBehaviour {

    public GameObject freeze_Block;
    public GameObject shield_Block;
    public Transform canvas;
    int spawnNum = 1;

    void SpawnFreeze()
    {
        for(int i=0; i<spawnNum; i++)
        {
            SpawnFreezeBlock();
        }
    }

    void SpawnShield()
    {
        for (int i = 0; i < spawnNum; i++)
        {
            SpawnShieldBlock();
        }
    }

    // Use this for initialization
    void Start () {
        SpawnFreeze();
        SpawnShield();
		
	}

    public void SpawnOneBlockAfterDelay(int delaySeconds)
    {
        Invoke("SpawnFreezeBlock", delaySeconds);
        Invoke("SpawnShieldBlock", delaySeconds);
    }

    public void SpawnFreezeBlock()
    {
        Vector3 blockPos = new Vector3(canvas.position.x + Random.Range(-5.0f, 7.5f),
                                       canvas.position.y + Random.Range(2.6f, 2.6f),
                                        canvas.position.z + Random.Range(6.5f, -6.6f));
        Instantiate(freeze_Block, blockPos, Quaternion.identity);
    }

    public void SpawnShieldBlock()
    {
        Vector3 blockPos = new Vector3(canvas.position.x + Random.Range(-5.0f, 7.5f),
                               canvas.position.y + Random.Range(2.6f, 2.6f),
                                canvas.position.z + Random.Range(6.5f, -6.6f));
        Instantiate(shield_Block, blockPos, Quaternion.identity);
    }

}

```