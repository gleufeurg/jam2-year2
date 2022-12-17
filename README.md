# jam2-year2


//PlayerController Script



using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

[RequireComponent(typeof(BoxCollider2D))]
[RequireComponent(typeof(Rigidbody2D))]
public class PlayerController : MonoBehaviour
{
    #region Variables

    [Header("Stats")]
    public float speed;
    public float swordDmg;
    public float swordActiveFrames;

    public float spawnTime;
    public float aps;

    [Space(10f)]

    [Header("Items count")]
    public int swordCount;
    public int swordNeeded;

    [Space(10f)]

    [Header("Refs")]
    [SerializeField] private GameObject swordprefab;
    [SerializeField] private GameObject anvilPrefab;
    [SerializeField] private GameObject kamera;
    [SerializeField] private Transform spawnPos;
    [SerializeField] private Transform anvilSpawnPos;

    [Space(10f)]

    [Header("Others")]
    [SerializeField] private bool canAttack = true;
    [SerializeField] private bool canSpawn = true;
    [SerializeField] private GameObject[] ennemySpawns;

    public List<string> items;

    [Space(10f)]

    [SerializeField] private Rigidbody2D rb;

    #endregion

    void Start()
    {
        rb = GetComponent<Rigidbody2D>();

        items = new List<string>();
    }

    void Update()
    {
        Move();
        Shoot(aps);
        SpawnEnnemy(spawnTime);
    }

    private void SpawnEnnemy(float _spawnTime)
    {
        if (canSpawn == false)
            return;

        StartCoroutine(EnnemySpawnCoroutine(_spawnTime));
    }

    //player Auto Attack
    private void Shoot(float _aps)
    {
        if (canAttack == false)
            return;        

        StartCoroutine(ShootCoroutine(_aps));
    }

    private void Move()
    {
        float velocityX = Input.GetAxisRaw("Horizontal") * speed;
        float velocityY = Input.GetAxisRaw("Vertical") * speed;

        Vector3 velocity = new Vector2(velocityX, velocityY);

        rb.velocity = velocity;
    }

    //Player pick a collectable object, interact with an anvil or take damage from bullet or ennemies
    private void OnTriggerEnter2D(Collider2D collision)
    {
        if (collision.CompareTag("Collectable"))
        {
            string itemType = collision.GetComponent<CollectableType>().type;
            items.Add(itemType);

            swordCount++;

            Debug.Log("You collected a : " + itemType);
            Debug.Log("You have " + items.Count);

            if (swordCount == swordNeeded)
            {
                var spawnAnvil = Instantiate(anvilPrefab, anvilSpawnPos.position, Quaternion.identity);
                Debug.Log("Une enclume est apparu");
            }
        }
        else if (collision.CompareTag("Interractable"))
        {
            if (swordCount >= swordNeeded)
            {
                Debug.Log("Lezgoooo !!! Your sword has been upgraded");
            }
            else
            {
                Debug.Log("oooooohhh...");
                return;
            }
        }
        Destroy(collision.gameObject);
    }

    //Attack coroutine
    IEnumerator ShootCoroutine(float _shootCadence)
    {
        canAttack = false;

        StartCoroutine(ActiveFramesCoroutine(swordActiveFrames));

        Debug.Log("Commence à tirer");
        yield return new WaitForSeconds(_shootCadence);
        Debug.Log("Finit de tirer");

        canAttack = true;
    }
    
    //Active frames coroutine
    IEnumerator ActiveFramesCoroutine(float _activetime)
    {
        GameObject swordAttack = Instantiate(swordprefab, spawnPos.position, Quaternion.identity, spawnPos);

        yield return new WaitForSeconds(_activetime);

        Destroy(swordAttack);
    }
    
    IEnumerator EnnemySpawnCoroutine(float _spawntime)
    {
        canSpawn = false;

        var rdm = UnityEngine.Random.Range(0, 3);
        GameObject spawnChoosen = ennemySpawns[rdm];

        Instantiate(spawnChoosen, spawnChoosen.transform.position, Quaternion.identity);

        Debug.Log("Spawn Ennemy");
        yield return new WaitForSeconds(_spawntime);

        canSpawn = true;
    }
}


//
//
//
//
//Pour collecter un item, il faut ce script




using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CollectableType : MonoBehaviour
{
    public string type;
}



// le placer dans sword et ajouter le tag collectable
