# cape_to_bluff_mobile_full_final.py

import random, math
from kivy.app import App
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.label import Label
from kivy.uix.button import Button
from kivy.uix.progressbar import ProgressBar
from kivy.clock import Clock
from kivy.uix.popup import Popup
from kivy.core.audio import SoundLoader
from kivy.graphics import Color, Line, Ellipse, PushMatrix, PopMatrix, Rotate

TOTAL_DISTANCE_KM = 3113
TOTAL_DAYS = 34

# 34 stages
STAGES = [
    ("Cape Reinga → Ahipara", 94), ("Ahipara → Kaitaia", 90),
    ("Kaitaia → Kawakawa", 92), ("Kawakawa → Whangārei", 95),
    ("Whangārei → Mangawhai Heads", 92), ("Mangawhai Heads → Orewa", 91),
    ("Orewa → Manukau", 90), ("Manukau → Hamilton", 96),
    ("Hamilton → REST DAY", 0), ("Hamilton → Putāruru", 90),
    ("Putāruru → Taupō", 94), ("Taupō → Taihape", 96),
    ("Taihape → Palmerston North", 92), ("Palmerston North → Levin", 90),
    ("Levin → Wellington", 94), ("Ferry to Picton → Blenheim", 90),
    ("Blenheim → Nelson", 94), ("REST DAY in Nelson", 0),
    ("Nelson → Westport", 98), ("Westport → Greymouth", 94),
    ("Greymouth → Hokitika", 92), ("Hokitika → Franz Josef", 94),
    ("Franz Josef → Haast", 98), ("Haast → Wanaka", 96),
    ("REST DAY in Wanaka", 0), ("Wanaka → Cromwell", 92),
    ("Cromwell → Roxburgh", 94), ("Roxburgh → Ranfurly", 90),
    ("Ranfurly → Middlemarch", 91), ("Middlemarch → Dunedin", 94),
    ("Dunedin → Balclutha", 92), ("Balclutha → Gore", 94),
    ("Gore → Invercargill", 90), ("Invercargill → Bluff", 91),
]

# Wheel of Doom challenges
WHEEL_OF_DOOM_CHALLENGES = [
    ("🚫 Silent Ride — No music", {"energy": -5, "morale": -5}),
    ("🥵 Hardest Gear Hill", {"energy": -10, "morale": -2}),
    ("⏱️ Blind Navigator", {"energy": -8, "morale": -5}),
    ("➕ +10 km Extra", {"distance": 10, "energy": -5}),
    ("🎥 Livestream Mode", {"energy": -7, "morale": -3}),
    ("🚴 Out of Saddle", {"energy": -6}),
    ("🛑 Non-Stop 30", {"energy": -12, "morale": -4}),
    ("🌧️ Rain Jacket Mode", {"energy": -4}),
    ("📸 Photo Frenzy", {"energy": -3, "morale": -2}),
    ("😎 Dress-Up Mode", {"energy": -2, "morale": -1}),
    ("🥵 Town Crusher", {"energy": -15, "morale": -5}),
]

# Map visualization
class MapBox(BoxLayout):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.bind(size=self.update_canvas, pos=self.update_canvas)
        self.marker_pos = 0

    def update_marker(self, progress_fraction):
        self.marker_pos = progress_fraction * self.width
        self.update_canvas()

    def update_canvas(self, *args):
        self.canvas.clear()
        with self.canvas:
            Color(0.6,0.6,0.6)
            Line(points=[50,self.center_y,self.width-50,self.center_y], width=4)
            stage_count=len(STAGES)
            for i in range(stage_count):
                x=50+i*(self.width-100)/(stage_count-1)
                Color(0.7,0.7,0.7)
                Ellipse(pos=(x-4,self.center_y-4), size=(8,8))
            Color(1,0,0)
            Ellipse(pos=(50+self.marker_pos-8,self.center_y-12), size=(16,16))

# Wheel of Doom visualization
class WheelWidget(BoxLayout):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.rotation_angle=0
        self.spinning=False
        self.bind(size=self.draw_wheel,pos=self.draw_wheel)

    def draw_wheel(self, *args):
        self.canvas.clear()
        with self.canvas:
            PushMatrix()
            self.rot = Rotate(angle=self.rotation_angle, origin=self.center)
            angle_per = 360/len(WHEEL_OF_DOOM_CHALLENGES)
            for i, (name, _) in enumerate(WHEEL_OF_DOOM_CHALLENGES):
                Color(random.random(), random.random(), random.random())
                Ellipse(pos=(self.center_x-100,self.center_y-100), size=(200,200),
                        angle_start=i*angle_per, angle_end=(i+1)*angle_per)
            PopMatrix()
            Color(1,0,0)
            Line(points=[self.center_x,self.center_y+110,self.center_x,self.center_y+150], width=3)

    def spin_wheel(self, callback):
        if self.spinning: return
        self.spinning=True
        self.target_angle=random.randint(720,1440)
        self.callback=callback
        Clock.schedule_interval(self.update_spin,0.02)

    def update_spin(self, dt):
        if self.target_angle<=0:
            self.spinning=False
            Clock.unschedule(self.update_spin)
            sector=int(((360-(self.rotation_angle%360))/(360/len(WHEEL_OF_DOOM_CHALLENGES)))%len(WHEEL_OF_DOOM_CHALLENGES))
            challenge,effects=WHEEL_OF_DOOM_CHALLENGES[sector]
            self.callback(challenge,effects)
            return False
        step=max(5,self.target_angle/20)
        self.rotation_angle+=step
        self.target_angle-=step
        self.draw_wheel()

# Main app
class CapeToBluffMobile(App):
    def build(self):
        self.current_day=1
        self.stage_index=0
        self.distance_today=0.0
        self.total_distance_done=0.0
        self.energy=100
        self.morale=100
        self.game_over=False

        # Load sounds (add your sound files to 'sounds/' folder)
        self.sounds={
            "pedal": SoundLoader.load("sounds/pedal.wav"),
            "wheel": SoundLoader.load("sounds/wheel.wav"),
            "eat": SoundLoader.load("sounds/eat.wav"),
            "rest": SoundLoader.load("sounds/rest.wav"),
            "event": SoundLoader.load("sounds/event.wav"),
        }

        self.root=BoxLayout(orientation="vertical",spacing=8,padding=12)
        self.map_box=MapBox(size_hint=(1,0.25))
        self.root.add_widget(self.map_box)

        self.status=Label(text=self.get_status_text(), font_size=18)
        self.root.add_widget(self.status)

        self.message=Label(text="Ready to ride!", font_size=16)
        self.root.add_widget(self.message)

        self.daily_bar=ProgressBar(max=STAGES[self.stage_index][1])
        self.total_bar=ProgressBar(max=TOTAL_DISTANCE_KM)
        self.root.add_widget(Label(text="Daily Progress:"))
        self.root.add_widget(self.daily_bar)
        self.root.add_widget(Label(text="Total Progress:"))
        self.root.add_widget(self.total_bar)

        self.pedal_btn=Button(text="🚴 Pedal", font_size=28)
        self.pedal_btn.bind(on_press=self.on_pedal)
        self.root.add_widget(self.pedal_btn)

        self.spin_btn=Button(text="🎯 Spin Wheel of Doom", font_size=22)
        self.spin_btn.bind(on_press=self.on_spin)
        self.root.add_widget(self.spin_btn)

        self.rest_btn=Button(text="🛌 Rest", font_size=18)
        self.rest_btn.bind(on_press=self.on_rest)
        self.root.add_widget(self.rest_btn)

        self.eat_btn=Button(text="🍌 Eat", font_size=18)
        self.eat_btn.bind(on_press=self.on_eat)
        self.root.add_widget(self.eat_btn)

        # Wheel visualization
        self.wheel=WheelWidget(size_hint=(1,0.3))
        self.root.add_widget(self.wheel)

        Clock.schedule_interval(self.background_tick,5)
        return self.root

    # ------------------ Utility functions ------------------
    def get_status_text(self):
        _,stage_km=STAGES[self.stage_index]
        return f"Day {self.current_day}/{TOTAL_DAYS} | Today: {self.distance_today:.1f}/{stage_km} km | Total: {self.total_distance_done:.1f}/{TOTAL_DISTANCE_KM} km | Energy: {self.energy} | Morale: {self.morale}"

    def update_map(self):
        self.daily_bar.value=self.distance_today
        self.total_bar.value=self.total_distance_done
        progress_fraction=min(self.total_distance_done/TOTAL_DISTANCE_KM,1.0)
        self.map_box.update_marker(progress_fraction)
        self.status.text=self.get_status_text()

    def show_notification(self, title, message):
        content=BoxLayout(orientation='vertical')
        content.add_widget(Label(text=message))
        btn=Button(text='OK', size_hint=(1,0.3))
        content.add_widget(btn)
        popup=Popup(title=title, content=content, size_hint=(0.8,0.4))
        btn.bind(on_release=popup.dismiss)
        popup.open()

    # ------------------ Game Actions ------------------
    def on_pedal(self, instance):
        if self.game_over: return
        if self.sounds["pedal"]: self.sounds["pedal"].play()
        stage_name, stage_km = STAGES[self.stage_index]
        if stage_km==0:
            self.show_rest_day()
            return
        base=12
        weather=random.choices(["Sunny","Headwind","Rain","Tailwind"],weights=[50,20,20,10])[0]
        if weather=="Headwind": base*=0.6; self.energy-=10
        elif weather=="Rain": base*=0.8; self.energy-=8
        elif weather=="Tailwind": base*=1.2; self.energy-=5
        else: self.energy-=7
        if self.morale<40: base*=0.85
        self.distance_today+=base
        self.total_distance_done+=base
        # Random events
        if random.randint(1,15)==1: self.trigger_random_event()
        if random.randint(1,10)==1: self.trigger_wheel_of_doom()
        self.message.text=f"{weather} — rode {base:.1f} km"
        if self.distance_today>=stage_km: self.advance_stage()
        self.update_map()

    def on_rest(self, instance):
        if self.game_over: return
        if self.sounds["rest"]: self.sounds["rest"].play()
        self.energy=min(100,self.energy+20)
        self.morale=min(100,self.morale+15)
        self.message.text="Rested!"
        self.update_map()

    def on_eat(self, instance):
        if self.game_over: return
        if self.sounds["eat"]: self.sounds["eat"].play()
        self.energy=min(100,self.energy+25)
        self.message.text="Ate food!"
        self.update_map()

    def on_spin(self, instance):
        self.wheel.spin_wheel(self.apply_wheel_effects)

    # ------------------ Wheel / Random events ------------------
    def apply_wheel_effects(self, challenge, effects):
        if self.sounds["wheel"]: self.sounds["wheel"].play()
        self.show_notification("🎯 Wheel of Doom!", f"{challenge}")
        if "energy" in effects: self.energy=max(0,self.energy+effects["energy"])
        if "morale" in effects: self.morale=max(0,self.morale+effects["morale"])
        if "distance" in effects: self.distance_today+=effects["distance"]; self.total_distance_done+=effects["distance"]
        self.update_map()

    def trigger_wheel_of_doom(self):
        challenge,effects=random.choice(WHEEL_OF_DOOM_CHALLENGES)
        self.apply_wheel_effects(challenge,effects)

    def trigger_random_event(self):
        roll=random.randint(1,5)
        if roll==1: self.energy-=15; self.morale-=10; msg="🔧 Flat tire! Energy and morale down."
        elif roll==2: boost=20; self.distance_today+=boost; self.total_distance_done+=boost; msg=f"🍃 Tailwind bonus +{boost} km!"
        elif roll==3: self.morale+=15; msg="🎶 Scenic views boost morale!"
        else: self.energy-=8; msg="💨 Crosswind slowed you."
        if self.sounds["event"]: self.sounds["event"].play()
        self.show_notification("Random Event!", msg)
        self.update_map()

    # ------------------ Stage Management ------------------
    def show_rest_day(self):
        self.show_notification("🌴 Rest Day", "Take a break today! Energy and morale recover faster.")
        self.energy=min(100,self.energy+30)
        self.morale=min(100,self.morale+25)
        self.update_map()

    def advance_stage(self):
        self.stage_index+=1
        self.distance_today=0
        self.current_day+=1
        self.energy=min(100,self.energy+15)
        self.morale=min(100,self.morale+10)
        if self.stage_index>=len(STAGES): self.win_game()
        else: self.update_map()

    # ------------------ Background tick ------------------
    def background_tick(self, dt):
        if self.game_over: return
        self.energy-=1
        self.morale-=0.5
        if self.energy<=0: self.lose_game("Exhaustion")
        if self.morale<=0: self.lose_game("Lost morale")

    def win_game(self):
        self.show_notification("🏆 Victory!", f"Reached Bluff! Total ~{self.total_distance_done:.1f} km")
        self.game_over=True

    def lose_game(self,reason):
        self.show_notification("💀 Game Over", reason)
        self.game_over=True

if __name__=="__main__":
    CapeToBluffMobile().run()