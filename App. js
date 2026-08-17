import React, { useEffect, useState } from "react";
import {
  SafeAreaView,
  View,
  Text,
  TextInput,
  TouchableOpacity,
  ScrollView,
  StyleSheet,
  Alert,
  ActivityIndicator,
} from "react-native";
const SUPABASE_URL = "https://lkqlvtoxeowmayywirrz.supabase.co";
const SUPABASE_KEY = "sb_publishable_PXG5WU-vEVCIDkJfv_PXkA_U0ROQFC2";
const ADMIN_EMAIL = "abubakarwasali7@gmail.com";
const FALLBACK_SERVICES = [
  ["🪚", "Carpentry Works"],
  ["🚰", "Plumbing"],
  ["⚡", "Electrical Works"],
  ["🎨", "Painting"],
  ["🧱", "Masonry Works"],
  ["🔲", "Tiling"],
  ["🔩", "Welding & Metal Works"],
  ["🪟", "Aluminium & Glass"],
  ["🏗️", "POP & Ceiling Works"],
  ["❄️", "AC Installation & Repair"],
  ["🧊", "Refrigerator & Freezer"],
  ["🚪", "Door & Lock Works"],
  ["🏠", "Roofing Works"],
  ["🧹", "Cleaning Services"],
  ["🌳", "Landscaping & Gardening"],
  ["🚿", "Water Tank & Systems"],
  ["🛠️", "General Home Maintenance"],
];
export default function App() {
  const [screen, setScreen] = useState("auth");
  const [authMode, setAuthMode] = useState("login");
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [fullName, setFullName] = useState("");
  const [session, setSession] = useState(null);
  const [loading, setLoading] = useState(false);
  const [services, setServices] = useState([]);
  const [selectedService, setSelectedService] = useState(null);
  const [title, setTitle] = useState("");
  const [description, setDescription] = useState("");
  const [location, setLocation] = useState("");
  const [requests, setRequests] = useState([]);
  const [workers, setWorkers] = useState([]);
  const [workerName, setWorkerName] = useState("");
  const [workerPhone, setWorkerPhone] = useState("");
  const [workerLocation, setWorkerLocation] = useState("");
  const [workerService, setWorkerService] = useState("");
  async function api(path, options = {}) {
    const response = await fetch(`${SUPABASE_URL}${path}`, {
      ...options,
      headers: {
        apikey: SUPABASE_KEY,
        "Content-Type": "application/json",
        ...(options.headers || {}),
      },
    });
    const text = await response.text();
    let data = null;
    try {
      data = text ? JSON.parse(text) : null;
    } catch {
      data = text;
    }
    if (!response.ok) {
      throw new Error(
        data?.message ||
          data?.error_description ||
          data?.error ||
          data?.hint ||
          `Request failed (${response.status})`
      );
    }
    return data;
  }
  function authHeaders(token) {
    return {
      apikey: SUPABASE_KEY,
      Authorization: `Bearer ${
        token || session?.access_token || SUPABASE_KEY
      }`,
    };
  }
  async function login() {
    if (!email.trim() || !password) {
      Alert.alert("Error", "Enter email and password.");
      return;
    }
    setLoading(true);
    try {
      const data = await api("/auth/v1/token?grant_type=password", {
        method: "POST",
        headers: {
          Authorization: `Bearer ${SUPABASE_KEY}`,
        },
        body: JSON.stringify({
          email: email.trim(),
          password,
        }),
      });
      setSession(data);
      const isAdmin =
        data?.user?.email?.toLowerCase() ===
        ADMIN_EMAIL.toLowerCase();
      if (isAdmin) {
        setScreen("admin");
        await loadAdminData(data.access_token);
      } else {
        setScreen("home");
        await loadServices();
        await loadMyRequests(data.access_token);
      }
    } catch (error) {
      Alert.alert("Login failed", error.message);
    } finally {
      setLoading(false);
    }
  }
  async function signup() {
    if (!fullName.trim() || !email.trim() || !password) {
      Alert.alert("Error", "Fill all fields.");
      return;
    }
    if (password.length < 6) {
      Alert.alert(
        "Password",
        "Password must contain at least 6 characters."
      );
      return;
    }
    setLoading(true);
    try {
      const data = await api("/auth/v1/signup", {
        method: "POST",
        headers: {
          Authorization: `Bearer ${SUPABASE_KEY}`,
        },
        body: JSON.stringify({
          email: email.trim(),
          password,
          data: {
            full_name: fullName.trim(),
          },
        }),
      });
      if (data?.access_token) {
        setSession(data);
        setScreen("home");
        await loadServices();
        await loadMyRequests(data.access_token);
      } else {
        Alert.alert(
          "Account created",
          "Check your email, confirm it, then login."
        );
        setAuthMode("login");
      }
    } catch (error) {
      Alert.alert("Signup failed", error.message);
    } finally {
      setLoading(false);
    }
  }
  async function loadServices() {
    try {
      const data = await api(
        "/rest/v1/services?select=*&is_active=eq.true&order=id.asc",
        {
          headers: authHeaders(),
        }
      );
      setServices(data || []);
    } catch (error) {
      console.log("Services:", error.message);
      setServices([]);
    }
  }
  async function loadMyRequests(token) {
    if (!token) return;
    try {
      const data = await api(
        "/rest/v1/service_requests?select=*,services(name),job_assignments(id,status,assigned_at,workers(id,full_name,phone,location,service))&order=created_at.desc",
        {
          headers: authHeaders(token),
        }
      );
      setRequests(data || []);
    } catch (error) {
      console.log("My requests:", error.message);
      try {
        const fallback = await api(
          "/rest/v1/service_requests?select=*,services(name)&order=created_at.desc",
          {
            headers: authHeaders(token),
          }
        );
        setRequests(fallback || []);
      } catch (secondError) {
        console.log(
          "Requests fallback:",
          secondError.message
        );
      }
    }
  }
  async function loadAdminData(token) {
    if (!token) return;
    const h = authHeaders(token);
    try {
      const requestData = await api(
        "/rest/v1/service_requests?select=*,services(name),job_assignments(id,status,assigned_at,workers(id,full_name,phone,location,service))&order=created_at.desc",
        {
          headers: h,
        }
      );
      setRequests(requestData || []);
    } catch (error) {
      console.log("Admin requests:", error.message);
      try {
        const fallbackRequests = await api(
          "/rest/v1/service_requests?select=*,services(name)&order=created_at.desc",
          {
            headers: h,
          }
        );
        setRequests(fallbackRequests || []);
      } catch (e) {
        console.log(
          "Request fallback:",
          e.message
        );
      }
    }
    try {
      const workerData = await api(
        "/rest/v1/workers?select=*&order=created_at.desc",
        {
          headers: h,
        }
      );
      setWorkers(workerData || []);
    } catch (error) {
      console.log("Workers:", error.message);
      setWorkers([]);
    }
  }
  async function createWorker() {
    const name = workerName.trim();
    const phone = workerPhone.trim();
    const loc = workerLocation.trim();
    const service = workerService.trim();
    if (!name || !phone || !loc) {
      Alert.alert(
        "Missing information",
        "Enter worker name, phone and location."
      );
      return;
    }
    if (!session?.access_token) {
      Alert.alert(
        "Login Required",
        "Please login as admin again."
      );
      return;
    }
    setLoading(true);
    try {
      await api("/rest/v1/workers", {
        method: "POST",
        headers: {
          ...authHeaders(session.access_token),
          Prefer: "return=representation",
        },
        body: JSON.stringify({
          full_name: name,
          phone,
          location: loc,
          service: service || null,
          is_available: true,
        }),
      });
      setWorkerName("");
      setWorkerPhone("");
      setWorkerLocation("");
      setWorkerService("");
      await loadAdminData(session.access_token);
      Alert.alert(
        "Success",
        "Worker added successfully."
      );
      setScreen("admin");
    } catch (error) {
      Alert.alert("Worker Error", error.message);
    } finally {
      setLoading(false);
    }
  }
  async function deleteWorker(id) {
    Alert.alert(
      "Delete Worker",
      "Are you sure you want to delete this worker?",
      [
        {
          text: "Cancel",
          style: "cancel",
        },
        {
          text: "Delete",
          style: "destructive",
          onPress: async () => {
            setLoading(true);
            try {
              try {
                await api(
                  `/rest/v1/job_assignments?worker_id=eq.${id}`,
                  {
                    method: "DELETE",
                    headers: {
                      ...authHeaders(
                        session?.access_token
                      ),
                      Prefer: "return=minimal",
                    },
                  }
                );
              } catch (assignmentError) {
                console.log(
                  "Assignment delete:",
                  assignmentError.message
                );
              }
              await api(
                `/rest/v1/workers?id=eq.${id}`,
                {
                  method: "DELETE",
                  headers: {
                    ...authHeaders(
                      session?.access_token
                    ),
                    Prefer: "return=minimal",
                  },
                }
              );
              await loadAdminData(
                session.access_token
              );
              Alert.alert(
                "Success",
                "Worker deleted."
              );
            } catch (error) {
              Alert.alert(
                "Delete Worker Error",
                error.message
              );
            } finally {
              setLoading(false);
            }
          },
        },
      ]
    );
  }
  async function assignWorker(request) {
    if (!session?.access_token) return;
    setLoading(true);
    try {
      const existingAssignments =
        request.job_assignments || [];
      if (existingAssignments.length > 0) {
        Alert.alert(
          "Already Assigned",
          "This request already has a worker assigned."
        );
        return;
      }
      const availableWorkers = workers.filter(
        (worker) =>
          worker.is_available === true
      );
      if (availableWorkers.length === 0) {
        Alert.alert(
          "No Available Worker",
          "There is currently no available worker."
        );
        return;
      }
      const requestService =
        request.services?.name?.toLowerCase() || "";
      let selectedWorker =
        availableWorkers.find((worker) => {
          const workerService =
            worker.service?.toLowerCase() || "";
          return (
            workerService &&
            requestService &&
            (workerService.includes(
              requestService
            ) ||
              requestService.includes(
                workerService
              ))
          );
        });
      if (!selectedWorker) {
        selectedWorker = availableWorkers[0];
      }
      await api(
        "/rest/v1/job_assignments",
        {
          method: "POST",
          headers: {
            ...authHeaders(
              session.access_token
            ),
            Prefer: "return=representation",
          },
          body: JSON.stringify({
            request_id: request.id,
            worker_id: selectedWorker.id,
            status: "assigned",
            assigned_at:
              new Date().toISOString(),
          }),
        }
      );
      await api(
        `/rest/v1/service_requests?id=eq.${request.id}`,
        {
          method: "PATCH",
          headers: {
            ...authHeaders(
              session.access_token
            ),
            Prefer: "return=representation",
          },
          body: JSON.stringify({
            status: "accepted",
            updated_at:
              new Date().toISOString(),
          }),
        }
      );
      await api(
        `/rest/v1/workers?id=eq.${selectedWorker.id}`,
        {
          method: "PATCH",
          headers: {
            ...authHeaders(
              session.access_token
            ),
            Prefer: "return=representation",
          },
          body: JSON.stringify({
            is_available: false,
          }),
        }
      );
      await loadAdminData(
        session.access_token
      );
      Alert.alert(
        "Success",
        `${selectedWorker.full_name} has been assigned successfully.`
      );
    } catch (error) {
      Alert.alert(
        "Assignment Error",
        error.message
      );
    } finally {
      setLoading(false);
    }
  }
  async function updateRequestStatus(
    requestId,
    status
  ) {
    if (!session?.access_token) return;
    setLoading(true);
    try {
      await api(
        `/rest/v1/service_requests?id=eq.${requestId}`,
        {
          method: "PATCH",
          headers: {
            ...authHeaders(
              session.access_token
            ),
            Prefer: "return=representation",
          },
          body: JSON.stringify({
            status,
            updated_at:
              new Date().toISOString(),
          }),
        }
      );
      if (status === "completed") {
        const request = requests.find(
          (item) =>
            String(item.id) ===
            String(requestId)
        );
        const assignments =
          request?.job_assignments || [];
        for (const assignment of assignments) {
          if (assignment?.workers?.id) {
            try {
              await api(
                `/rest/v1/workers?id=eq.${assignment.workers.id}`,
                {
                  method: "PATCH",
                  headers: {
                    ...authHeaders(
                      session.access_token
                    ),
                    Prefer: "return=representation",
                  },
                  body: JSON.stringify({
                    is_available: true,
                  }),
                }
              );
            } catch (workerError) {
              console.log(
                "Worker availability:",
                workerError.message
              );
            }
          }
        }
      }
      await loadAdminData(
        session.access_token
      );
      Alert.alert(
        "Success",
        `Request is now ${status}.`
      );
    } catch (error) {
      Alert.alert(
        "Update Error",
        error.message
      );
    } finally {
      setLoading(false);
    }
  }
  async function deleteCompletedRequest(
    request
  ) {
    if (!session?.access_token) return;
    if (
      String(request.status).toLowerCase() !==
      "completed"
    ) {
      Alert.alert(
        "Cannot Delete",
        "Only completed service requests can be deleted."
      );
      return;
    }
    Alert.alert(
      "Delete Completed Job",
      "Are you sure you want to remove this completed job from your account?",
      [
        {
          text: "Cancel",
          style: "cancel",
        },
        {
          text: "Delete",
          style: "destructive",
          onPress: async () => {
            setLoading(true);
            try {
              try {
                await api(
                  `/rest/v1/job_assignments?request_id=eq.${request.id}`,
                  {
                    method: "DELETE",
                    headers: {
                      ...authHeaders(
                        session.access_token
                      ),
                      Prefer: "return=minimal",
                    },
                  }
                );
              } catch (assignmentError) {
                console.log(
                  "Assignment cleanup:",
                  assignmentError.message
                );
              }
              await api(
                `/rest/v1/service_requests?id=eq.${request.id}`,
                {
                  method: "DELETE",
                  headers: {
                    ...authHeaders(
                      session.access_token
                    ),
                    Prefer: "return=minimal",
                  },
                }
              );
              await loadMyRequests(
                session.access_token
              );
              Alert.alert(
                "Deleted",
                "Completed job has been removed from your account."
              );
            } catch (error) {
              Alert.alert(
                "Delete Error",
                error.message
              );
            } finally {
              setLoading(false);
            }
          },
        },
      ]
    );
  }
  async function adminDeleteRequest(
    request
  ) {
    if (!session?.access_token) return;
    Alert.alert(
      "Delete Request",
      "Delete this service request permanently?",
      [
        {
          text: "Cancel",
          style: "cancel",
        },
        {
          text: "Delete",
          style: "destructive",
          onPress: async () => {
            setLoading(true);
            try {
              try {
                await api(
                  `/rest/v1/job_assignments?request_id=eq.${request.id}`,
                  {
                    method: "DELETE",
                    headers: {
                      ...authHeaders(
                        session.access_token
                      ),
                      Prefer: "return=minimal",
                    },
                  }
                );
              } catch (assignmentError) {
                console.log(
                  "Admin assignment cleanup:",
                  assignmentError.message
                );
              }
              await api(
                `/rest/v1/service_requests?id=eq.${request.id}`,
                {
                  method: "DELETE",
                  headers: {
                    ...authHeaders(
                      session.access_token
                    ),
                    Prefer: "return=minimal",
                  },
                }
              );
              await loadAdminData(
                session.access_token
              );
              Alert.alert(
                "Deleted",
                "Service request deleted successfully."
              );
            } catch (error) {
              Alert.alert(
                "Delete Error",
                error.message
              );
            } finally {
              setLoading(false);
            }
          },
        },
      ]
    );
  }
  async function createRequest() {
    if (!selectedService) {
      Alert.alert(
        "Service",
        "Choose a service."
      );
      return;
    }
    if (
      !title.trim() ||
      !description.trim() ||
      !location.trim()
    ) {
      Alert.alert(
        "Missing information",
        "Fill all fields."
      );
      return;
    }
    if (!session?.user?.id) {
      Alert.alert(
        "Login required",
        "Please login again."
      );
      return;
    }
    setLoading(true);
    try {
      await api(
        "/rest/v1/service_requests",
        {
          method: "POST",
          headers: {
            ...authHeaders(
              session.access_token
            ),
            Prefer: "return=representation",
          },
          body: JSON.stringify({
            customer_id:
              session.user.id,
            service_id:
              selectedService.id,
            title: title.trim(),
            description:
              description.trim(),
            location:
              location.trim(),
            status: "pending",
          }),
        }
      );
      setTitle("");
      setDescription("");
      setLocation("");
      setSelectedService(null);
      await loadMyRequests(
        session.access_token
      );
      Alert.alert(
        "Success",
        "Service request submitted."
      );
      setScreen("requests");
    } catch (error) {
      Alert.alert(
        "Request Error",
        error.message
      );
    } finally {
      setLoading(false);
    }
  }
  async function logout() {
    try {
      if (session?.access_token) {
        await fetch(
          `${SUPABASE_URL}/auth/v1/logout`,
          {
            method: "POST",
            headers: {
              apikey: SUPABASE_KEY,
              Authorization:
                `Bearer ${session.access_token}`,
            },
          }
        );
      }
    } catch {}
    setSession(null);
    setEmail("");
    setPassword("");
    setFullName("");
    setRequests([]);
    setWorkers([]);
    setSelectedService(null);
    setScreen("auth");
  }
  useEffect(() => {
    loadServices();
  }, []);
  if (screen === "auth") {
    return (
      <SafeAreaView style={styles.container}>
        <ScrollView
          contentContainerStyle={
            styles.authContainer
          }
        >
          <View style={styles.logo}>
            <Text style={styles.logoText}>
              A-T
            </Text>
          </View>
          <Text style={styles.appName}>
            A-T SERVICE REPAIR
          </Text>
          <Text style={styles.authSubtitle}>
            Professional services for homes,
            offices and buildings.
          </Text>
          <View style={styles.switchRow}>
            <TouchableOpacity
              style={
                authMode === "login"
                  ? styles.switchActive
                  : styles.switch
              }
              onPress={() =>
                setAuthMode("login")
              }
            >
              <Text style={styles.switchText}>
                Login
              </Text>
            </TouchableOpacity>
            <TouchableOpacity
              style={
                authMode === "signup"
                  ? styles.switchActive
                  : styles.switch
              }
              onPress={() =>
                setAuthMode("signup")
              }
            >
              <Text style={styles.switchText}>
                Sign Up
              </Text>
            </TouchableOpacity>
          </View>
          {authMode === "signup" && (
            <TextInput
              style={styles.input}
              placeholder="Full name"
              value={fullName}
              onChangeText={setFullName}
            />
          )}
          <TextInput
            style={styles.input}
            placeholder="Email address"
            keyboardType="email-address"
            autoCapitalize="none"
            value={email}
            onChangeText={setEmail}
          />
          <TextInput
            style={styles.input}
            placeholder="Password"
            secureTextEntry
            value={password}
            onChangeText={setPassword}
          />
          <TouchableOpacity
            style={styles.primaryButton}
            onPress={
              authMode === "login"
                ? login
                : signup
            }
            disabled={loading}
          >
            {loading ? (
              <ActivityIndicator
                color="#FFFFFF"
              />
            ) : (
              <Text style={styles.primaryText}>
                {authMode === "login"
                  ? "Login"
                  : "Create Account"}
              </Text>
            )}
          </TouchableOpacity>
        </ScrollView>
      </SafeAreaView>
    );
  }
  if (screen === "admin") {
    return (
      <SafeAreaView style={styles.container}>
        <ScrollView
          contentContainerStyle={
            styles.content
          }
        >
          <View style={styles.adminHeader}>
            <View>
              <Text style={styles.smallText}>
                ADMIN PANEL
              </Text>
              <Text style={styles.adminTitle}>
                A-T SERVICE REPAIR
              </Text>
            </View>
            <TouchableOpacity
              onPress={logout}
            >
              <Text style={styles.logout}>
                Logout
              </Text>
            </TouchableOpacity>
          </View>
          <View style={styles.statsRow}>
            <View style={styles.statCard}>
              <Text style={styles.statNumber}>
                {requests.length}
              </Text>
              <Text style={styles.statLabel}>
                Requests
              </Text>
            </View>
            <View style={styles.statCard}>
              <Text style={styles.statNumber}>
                {workers.length}
              </Text>
              <Text style={styles.statLabel}>
                Workers
              </Text>
            </View>
            <View style={styles.statCard}>
              <Text style={styles.statNumber}>
                {
                  requests.filter(
                    (r) =>
                      r.status ===
                      "completed"
                  ).length
                }
              </Text>
              <Text style={styles.statLabel}>
                Completed
              </Text>
            </View>
          </View>
          <TouchableOpacity
            style={styles.refreshButton}
            onPress={() =>
              loadAdminData(
                session?.access_token
              )
            }
          >
            <Text style={styles.refreshText}>
              🔄 Refresh Admin Data
            </Text>
          </TouchableOpacity>
          <TouchableOpacity
            style={styles.workerButton}
            onPress={() =>
              setScreen("addWorker")
            }
          >
            <Text
              style={styles.workerButtonText}
            >
              + Add Worker
            </Text>
          </TouchableOpacity>
          <Text style={styles.sectionTitle}>
            Workers
          </Text>
          {workers.length === 0 ? (
            <View style={styles.empty}>
              <Text
                style={styles.emptyText}
              >
                No workers yet.
              </Text>
            </View>
          ) : (
            workers.map((worker) => (
              <View
                style={styles.card}
                key={worker.id}
              >
                <Text
                  style={styles.cardTitle}
                >
                  {worker.full_name}
                </Text>
                <Text>
                  📞 {worker.phone}
                </Text>
                <Text>
                  📍 {worker.location}
                </Text>
                <Text>
                  🔧{" "}
                  {worker.service ||
                    "General Service"}
                </Text>
                <Text
                  style={
                    styles.availability
                  }
                >
                  {worker.is_available
                    ? "🟢 Available"
                    : "🔴 Unavailable"}
                </Text>
                <TouchableOpacity
                  style={
                    styles.deleteButton
                  }
                  onPress={() =>
                    deleteWorker(
                      worker.id
                    )
                  }
                >
                  <Text
                    style={
                      styles.deleteButtonText
                    }
                  >
                    Delete Worker
                  </Text>
                </TouchableOpacity>
              </View>
            ))
          )}
          <Text style={styles.sectionTitle}>
            Service Requests
          </Text>
          {requests.length === 0 ? (
            <View style={styles.empty}>
              <Text
                style={styles.emptyText}
              >
                No service requests yet.
              </Text>
            </View>
          ) : (
            requests.map((request) => (
              <View
                style={styles.card}
                key={request.id}
              >
                <Text style={styles.blue}>
                  {request.services?.name ||
                    "Service"}
                </Text>
                <Text
                  style={styles.cardTitle}
                >
                  {request.title}
                </Text>
                <Text
                  style={styles.description}
                >
                  {request.description}
                </Text>
                <Text
                  style={styles.location}
                >
                  📍 {request.location}
                </Text>
                <View style={styles.status}>
                  <Text
                    style={
                      styles.statusText
                    }
                  >
                    {String(
                      request.status
                    ).toUpperCase()}
                  </Text>
                </View>
                <TouchableOpacity
                  style={
                    styles.secondaryButton
                  }
                  onPress={() =>
                    assignWorker(
                      request
                    )
                  }
                  disabled={loading}
                >
                  <Text
                    style={
                      styles.secondaryText
                    }
                  >
                    👷 Assign Worker
                  </Text>
                </TouchableOpacity>
                <TouchableOpacity
                  style={
                    styles.secondaryButton
                  }
                  onPress={() =>
                    updateRequestStatus(
                      request.id,
                      "in_progress"
                    )
                  }
                  disabled={loading}
                >
                  <Text
                    style={
                      styles.secondaryText
                    }
                  >
                    🔧 In Progress
                  </Text>
                </TouchableOpacity>
                <TouchableOpacity
                  style={
                    styles.completeButton
                  }
                  onPress={() =>
                    updateRequestStatus(
                      request.id,
                      "completed"
                    )
                  }
                  disabled={loading}
                >
                  <Text
                    style={
                      styles.completeText
                    }
                  >
                    ✅ Complete
                  </Text>
                </TouchableOpacity>
                <TouchableOpacity
                  style={
                    styles.deleteRequestButton
                  }
                  onPress={() =>
                    adminDeleteRequest(
                      request
                    )
                  }
                  disabled={loading}
                >
                  <Text
                    style={
                      styles.deleteRequestText
                    }
                  >
                    🗑️ Delete Request
                  </Text>
                </TouchableOpacity>
              </View>
            ))
          )}
        </ScrollView>
      </SafeAreaView>
    );
  }
  if (screen === "addWorker") {
    return (
      <SafeAreaView style={styles.container}>
        <ScrollView
          contentContainerStyle={
            styles.content
          }
        >
          <TouchableOpacity
            onPress={() =>
              setScreen("admin")
            }
          >
            <Text style={styles.back}>
              ← Back
            </Text>
          </TouchableOpacity>
          <Text style={styles.title}>
            Add Worker
          </Text>
          <TextInput
            style={styles.input}
            placeholder="Worker full name"
            value={workerName}
            onChangeText={setWorkerName}
          />
          <TextInput
            style={styles.input}
            placeholder="Phone number"
            keyboardType="phone-pad"
            value={workerPhone}
            onChangeText={setWorkerPhone}
          />
          <TextInput
            style={styles.input}
            placeholder="Location"
            value={workerLocation}
            onChangeText={setWorkerLocation}
          />
          <TextInput
            style={styles.input}
            placeholder="Service e.g. Carpentry Works"
            value={workerService}
            onChangeText={setWorkerService}
          />
          <TouchableOpacity
            style={styles.primaryButton}
            onPress={createWorker}
            disabled={loading}
          >
            {loading ? (
              <ActivityIndicator
                color="#FFFFFF"
              />
            ) : (
              <Text style={styles.primaryText}>
                Save Worker
              </Text>
            )}
          </TouchableOpacity>
        </ScrollView>
      </SafeAreaView>
    );
  }
  if (screen === "request") {
    return (
      <SafeAreaView style={styles.container}>
        <ScrollView
          contentContainerStyle={
            styles.content
          }
        >
          <TouchableOpacity
            onPress={() =>
              setScreen("home")
            }
          >
            <Text style={styles.back}>
              ← Back
            </Text>
          </TouchableOpacity>
          <Text style={styles.title}>
            Request Service
          </Text>
          <View
            style={styles.selectedCard}
          >
            <Text style={styles.bigIcon}>
              {selectedService?.icon ||
                "🛠️"}
            </Text>
            <Text
              style={styles.selectedName}
            >
              {selectedService?.name ||
                "Service"}
            </Text>
          </View>
          <TextInput
            style={styles.input}
            placeholder="Job title"
            value={title}
            onChangeText={setTitle}
          />
          <TextInput
            style={[
              styles.input,
              styles.textArea,
            ]}
            placeholder="Describe the work"
            value={description}
            onChangeText={setDescription}
            multiline
          />
          <TextInput
            style={styles.input}
            placeholder="Service location"
            value={location}
            onChangeText={setLocation}
          />
          <TouchableOpacity
            style={styles.primaryButton}
            onPress={createRequest}
            disabled={loading}
          >
            {loading ? (
              <ActivityIndicator
                color="#FFFFFF"
              />
            ) : (
              <Text style={styles.primaryText}>
                Submit Service Request
              </Text>
            )}
          </TouchableOpacity>
        </ScrollView>
      </SafeAreaView>
    );
  }
  if (screen === "requests") {
    return (
      <SafeAreaView style={styles.container}>
        <ScrollView
          contentContainerStyle={
            styles.content
          }
        >
          <TouchableOpacity
            onPress={() =>
              setScreen("home")
            }
          >
            <Text style={styles.back}>
              ← Home
            </Text>
          </TouchableOpacity>
          <Text style={styles.title}>
            My Requests
          </Text>
          {requests.length === 0 ? (
            <View style={styles.empty}>
              <Text
                style={styles.emptyText}
              >
                You have no requests yet.
              </Text>
            </View>
          ) : (
            requests.map((request) => (
              <View
                style={styles.card}
                key={request.id}
              >
                <Text style={styles.blue}>
                  {request.services?.name ||
                    "Service"}
                </Text>
                <Text
                  style={styles.cardTitle}
                >
                  {request.title}
                </Text>
                <Text
                  style={styles.description}
                >
                  {request.description}
                </Text>
                <Text
                  style={styles.location}
                >
                  📍 {request.location}
                </Text>
                <View style={styles.status}>
                  <Text
                    style={
                      styles.statusText
                    }
                  >
                    {String(
                      request.status
                    ).toUpperCase()}
                  </Text>
                </View>
                {request.job_assignments &&
                  request
                    .job_assignments
                    .length > 0 && (
                    <View
                      style={
                        styles.workerInfo
                      }
                    >
                      <Text
                        style={
                          styles.workerInfoTitle
                        }
                      >
                        👷 Assigned Worker
                      </Text>
                      {request.job_assignments.map(
                        (assignment) => (
                          <View
                            key={
                              assignment.id
                            }
                            style={
                              styles.assignment
                            }
                          >
                            <Text
                              style={
                                styles.workerName
                              }
                            >
                              👤{" "}
                              {assignment
                                .workers
                                ?.full_name ||
                                "Worker"}
                            </Text>
                            <Text>
                              📞{" "}
                              {assignment
                                .workers
                                ?.phone ||
                                "No phone"}
                            </Text>
                            <Text>
                              📍{" "}
                              {assignment
                                .workers
                                ?.location ||
                                "No location"}
                            </Text>
                            <Text>
                              🔧{" "}
                              {assignment
                                .workers
                                ?.service ||
                                "General Service"}
                            </Text>
                            <View
                              style={
                                styles.assignmentStatus
                              }
                            >
                              <Text
                                style={
                                  styles.assignmentStatusText
                                }
                              >
                                📌{" "}
                                {String(
                                  assignment.status
                                ).toUpperCase()}
                              </Text>
                            </View>
                          </View>
                        )
                      )}
                    </View>
                  )}
                {String(
                  request.status
                ).toLowerCase() ===
                  "completed" && (
                  <TouchableOpacity
                    style={
                      styles.deleteRequestButton
                    }
                    onPress={() =>
                      deleteCompletedRequest(
                        request
                      )
                    }
                    disabled={loading}
                  >
                    <Text
                      style={
                        styles.deleteRequestText
                      }
                    >
                      🗑️ Delete Completed Job
                    </Text>
                  </TouchableOpacity>
                )}
              </View>
            ))
          )}
        </ScrollView>
      </SafeAreaView>
    );
  }
  return (
    <SafeAreaView style={styles.container}>
      <ScrollView
        contentContainerStyle={
          styles.content
        }
      >
        <View style={styles.homeHeader}>
          <View>
            <Text style={styles.smallText}>
              Welcome
            </Text>
            <Text style={styles.homeName}>
              {session?.user?.user_metadata
                ?.full_name ||
                session?.user?.email ||
                "Customer"}
            </Text>
          </View>
          <TouchableOpacity
            onPress={logout}
          >
            <Text style={styles.logout}>
              Logout
            </Text>
          </TouchableOpacity>
        </View>
        <View style={styles.hero}>
          <Text style={styles.heroTitle}>
            What service do you need?
          </Text>
          <Text style={styles.heroText}>
            Get professional workers for
            your home, office and building
            projects.
          </Text>
        </View>
        <TouchableOpacity
          style={styles.myRequestsButton}
          onPress={async () => {
            await loadMyRequests(
              session?.access_token
            );
            setScreen("requests");
          }}
        >
          <Text
            style={styles.myRequestsText}
          >
            📋 My Service Requests
          </Text>
        </TouchableOpacity>
        <Text style={styles.sectionTitle}>
          Our Services
        </Text>
        {(services.length
          ? services
          : FALLBACK_SERVICES.map(
              (item, index) => ({
                id: index + 1,
                icon: item[0],
                name: item[1],
              })
            )
        ).map((service) => (
          <TouchableOpacity
            key={service.id}
            style={styles.serviceCard}
            onPress={() => {
              setSelectedService(
                service
              );
              setScreen("request");
            }}
          >
            <Text style={styles.icon}>
              {service.icon || "🛠️"}
            </Text>
            <View
              style={styles.serviceInfo}
            >
              <Text
                style={styles.serviceTitle}
              >
                {service.name}
              </Text>
              <Text style={styles.tapText}>
                Request this service
              </Text>
            </View>
            <Text style={styles.arrow}>
              ›
            </Text>
          </TouchableOpacity>
        ))}
      </ScrollView>
    </SafeAreaView>
  );
}
const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: "#F5F7FA",
  },
  content: {
    padding: 20,
    paddingBottom: 70,
  },
  authContainer: {
    flexGrow: 1,
    justifyContent: "center",
    padding: 22,
  },
  logo: {
    width: 85,
    height: 85,
    borderRadius: 24,
    backgroundColor: "#111827",
    alignSelf: "center",
    justifyContent: "center",
    alignItems: "center",
    marginBottom: 18,
  },
  logoText: {
    color: "#FFFFFF",
    fontSize: 30,
    fontWeight: "900",
  },
  appName: {
    textAlign: "center",
    fontSize: 25,
    fontWeight: "900",
    color: "#111827",
  },
  authSubtitle: {
    textAlign: "center",
    color: "#6B7280",
    marginTop: 8,
    marginBottom: 25,
    lineHeight: 21,
  },
  switchRow: {
    flexDirection: "row",
    backgroundColor: "#E5E7EB",
    borderRadius: 14,
    padding: 4,
    marginBottom: 18,
  },
  switch: {
    flex: 1,
    padding: 12,
    alignItems: "center",
  },
  switchActive: {
    flex: 1,
    padding: 12,
    alignItems: "center",
    backgroundColor: "#111827",
    borderRadius: 11,
  },
  switchText: {
    color: "#374151",
    fontWeight: "800",
  },
  input: {
    backgroundColor: "#FFFFFF",
    borderWidth: 1,
    borderColor: "#E5E7EB",
    borderRadius: 14,
    padding: 16,
    fontSize: 16,
    marginBottom: 14,
  },
  primaryButton: {
    backgroundColor: "#111827",
    borderRadius: 14,
    padding: 17,
    alignItems: "center",
    marginTop: 5,
    marginBottom: 15,
  },
  primaryText: {
    color: "#FFFFFF",
    fontSize: 16,
    fontWeight: "900",
  },
  homeHeader: {
    flexDirection: "row",
    justifyContent: "space-between",
    alignItems: "center",
  },
  smallText: {
    color: "#6B7280",
  },
  homeName: {
    fontSize: 19,
    fontWeight: "800",
    color: "#111827",
    marginTop: 3,
  },
  logout: {
    color: "#DC2626",
    fontWeight: "900",
  },
  hero: {
    backgroundColor: "#111827",
    borderRadius: 22,
    padding: 22,
    marginTop: 22,
    marginBottom: 18,
  },
  heroTitle: {
    color: "#FFFFFF",
    fontSize: 24,
    fontWeight: "900",
    marginBottom: 8,
  },
  heroText: {
    color: "#D1D5DB",
    lineHeight: 21,
  },
  myRequestsButton: {
    backgroundColor: "#FFFFFF",
    borderRadius: 16,
    padding: 17,
    marginBottom: 25,
  },
  myRequestsText: {
    textAlign: "center",
    color: "#111827",
    fontWeight: "900",
  },
  sectionTitle: {
    fontSize: 22,
    fontWeight: "900",
    color: "#111827",
    marginBottom: 14,
    marginTop: 10,
  },
  serviceCard: {
    backgroundColor: "#FFFFFF",
    borderRadius: 17,
    padding: 16,
    marginBottom: 12,
    flexDirection: "row",
    alignItems: "center",
  },
  icon: {
    fontSize: 29,
    width: 48,
  },
  serviceInfo: {
    flex: 1,
  },
  serviceTitle: {
    fontSize: 16,
    fontWeight: "800",
    color: "#111827",
  },
  tapText: {
    color: "#6B7280",
    fontSize: 12,
    marginTop: 4,
  },
  arrow: {
    fontSize: 28,
    color: "#9CA3AF",
  },
  back: {
    color: "#2563EB",
    fontWeight: "900",
    fontSize: 16,
    marginBottom: 22,
  },
  title: {
    fontSize: 30,
    fontWeight: "900",
    color: "#111827",
    marginBottom: 20,
  },
  selectedCard: {
    backgroundColor: "#FFFFFF",
    borderRadius: 20,
    padding: 20,
    alignItems: "center",
    marginBottom: 18,
  },
  bigIcon: {
    fontSize: 50,
  },
  selectedName: {
    fontSize: 21,
    fontWeight: "900",
    marginTop: 10,
    color: "#111827",
  },
  textArea: {
    minHeight: 120,
    textAlignVertical: "top",
  },
  card: {
    backgroundColor: "#FFFFFF",
    borderRadius: 18,
    padding: 18,
    marginBottom: 15,
    borderWidth: 1,
    borderColor: "#E5E7EB",
  },
  cardTitle: {
    fontSize: 18,
    fontWeight: "900",
    color: "#111827",
    marginBottom: 7,
  },
  description: {
    color: "#4B5563",
    lineHeight: 20,
  },
  blue: {
    color: "#2563EB",
    fontWeight: "900",
    marginBottom: 7,
  },
  location: {
    marginTop: 10,
    color: "#6B7280",
  },
  status: {
    alignSelf: "flex-start",
    backgroundColor: "#E5E7EB",
    borderRadius: 10,
    paddingHorizontal: 10,
    paddingVertical: 7,
    marginTop: 12,
  },
  statusText: {
    fontSize: 11,
    fontWeight: "900",
    color: "#111827",
  },
  statsRow: {
    flexDirection: "row",
    gap: 10,
    marginBottom: 15,
  },
  statCard: {
    flex: 1,
    backgroundColor: "#FFFFFF",
    borderRadius: 15,
    padding: 14,
    alignItems: "center",
  },
  statNumber: {
    fontSize: 24,
    fontWeight: "900",
    color: "#111827",
  },
  statLabel: {
    color: "#6B7280",
    fontSize: 11,
    marginTop: 3,
  },
  adminHeader: {
    flexDirection: "row",
    justifyContent: "space-between",
    alignItems: "center",
    marginBottom: 20,
  },
  adminTitle: {
    fontSize: 20,
    fontWeight: "900",
    color: "#111827",
    marginTop: 4,
  },
  workerButton: {
    backgroundColor: "#111827",
    borderRadius: 14,
    padding: 16,
    alignItems: "center",
    marginBottom: 12,
  },
  workerButtonText: {
    color: "#FFFFFF",
    fontWeight: "900",
  },
  refreshButton: {
    backgroundColor: "#FFFFFF",
    borderRadius: 14,
    padding: 14,
    alignItems: "center",
    marginBottom: 12,
    borderWidth: 1,
    borderColor: "#E5E7EB",
  },
  refreshText: {
    color: "#111827",
    fontWeight: "900",
  },
  availability: {
    marginTop: 10,
    fontWeight: "800",
  },
  deleteButton: {
    backgroundColor: "#FEE2E2",
    borderRadius: 10,
    padding: 12,
    alignItems: "center",
    marginTop: 12,
  },
  deleteButtonText: {
    color: "#B91C1C",
    fontWeight: "900",
  },
  secondaryButton: {
    backgroundColor: "#EEF2FF",
    borderRadius: 10,
    padding: 13,
    alignItems: "center",
    marginTop: 10,
  },
  secondaryText: {
    color: "#3730A3",
    fontWeight: "900",
  },
  completeButton: {
    backgroundColor: "#DCFCE7",
    borderRadius: 10,
    padding: 13,
    alignItems: "center",
    marginTop: 10,
  },
  completeText: {
    color: "#166534",
    fontWeight: "900",
  },
  deleteRequestButton: {
    backgroundColor: "#FEE2E2",
    borderRadius: 10,
    padding: 13,
    alignItems: "center",
    marginTop: 10,
  },
  deleteRequestText: {
    color: "#B91C1C",
    fontWeight: "900",
  },
  workerInfo: {
    marginTop: 18,
    padding: 15,
    backgroundColor: "#F0FDF4",
    borderRadius: 14,
    borderWidth: 1,
    borderColor: "#BBF7D0",
  },
  workerInfoTitle: {
    fontSize: 17,
    fontWeight: "900",
    color: "#166534",
    marginBottom: 10,
  },
  assignment: {
    paddingTop: 5,
  },
  workerName: {
    fontSize: 16,
    fontWeight: "900",
    color: "#111827",
    marginBottom: 5,
  },
  assignmentStatus: {
    alignSelf: "flex-start",
    backgroundColor: "#DCFCE7",
    borderRadius: 8,
    paddingHorizontal: 9,
    paddingVertical: 6,
    marginTop: 8,
  },
  assignmentStatusText: {
    color: "#166534",
    fontWeight: "900",
    fontSize: 11,
  },
  empty: {
    backgroundColor: "#FFFFFF",
    padding: 25,
    borderRadius: 18,
  },
  emptyText: {
    color: "#6B7280",
    textAlign: "center",
  },
});
