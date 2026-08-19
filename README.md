# client-assignment
Full Name: Vamsi krishna Boggavarapu
Email: boggavarapuvamsikrishna1@gmail.com
Assignment Title:client-assignment
Date Started: 19/08/2026
Date Submitted: 20/08/2026



public class EventRecord {
    private String eventType;
    private String actorId;
    private String resourceType;
    private String resourceId;
    private Map<String, Object> payload;
    private long timestamp;
    private String currentHash;
    private String previousHash;

  
}



import java.security.MessageDigest;
import java.util.*;

@Service
public class AuditLogService {
    private List<EventRecord> records = new ArrayList<>();

    public EventRecord addEvent(EventRecord event) throws Exception {
        String prevHash = records.isEmpty() ? "GENESIS" : records.get(records.size()-1).getCurrentHash();
        event.setPreviousHash(prevHash);
        event.setCurrentHash(generateHash(event, prevHash));
        records.add(event);
        return event;
    }

    public List<EventRecord> queryEvents(String actorId, String resourceType, String resourceId, String eventType) {
        return records.stream()
                .filter(r -> (actorId == null || r.getActorId().equals(actorId)) &&
                             (resourceType == null || r.getResourceType().equals(resourceType)) &&
                             (resourceId == null || r.getResourceId().equals(resourceId)) &&
                             (eventType == null || r.getEventType().equals(eventType)))
                .toList();
    }

    public boolean verifyChain() throws Exception {
        for (int i = 1; i < records.size(); i++) {
            EventRecord current = records.get(i);
            String expectedHash = generateHash(current, current.getPreviousHash());
            if (!current.getCurrentHash().equals(expectedHash)) {
                return false; // tampering detected
            }
        }
        return true;
    }

    private String generateHash(EventRecord event, String prevHash) throws Exception {
        MessageDigest digest = MessageDigest.getInstance("SHA-256");
        String data = event.getEventType() + event.getActorId() + event.getResourceType() +
                      event.getResourceId() + event.getPayload().toString() +
                      event.getTimestamp() + prevHash;
        byte[] hash = digest.digest(data.getBytes("UTF-8"));
        return Base64.getEncoder().encodeToString(hash);
    }
}



@RestController
@RequestMapping("/audit")
public class AuditController {
    @Autowired
    private AuditLogService service;

    @PostMapping("/write")
    public EventRecord writeEvent(@RequestBody EventRecord event) throws Exception {
        return service.addEvent(event);
    }

    @GetMapping("/query")
    public List<EventRecord> queryEvents(@RequestParam(required=false) String actorId,
                                         @RequestParam(required=false) String resourceType,
                                         @RequestParam(required=false) String resourceId,
                                         @RequestParam(required=false) String eventType) {
        return service.queryEvents(actorId, resourceType, resourceId, eventType);
    }

    @GetMapping("/verify")
    public String verifyChain() throws Exception {
        return service.verifyChain() ? "Chain intact" : "Tampering detected!";
    }
}




